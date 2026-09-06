# Relier deux agents Hermes par A2A avec des tunnels SSH

Comment connecter deux agents Hermes sur des machines différentes via le
protocole A2A v1.0, sans ouvrir aucun port sur le pare-feu, et permettre une
initiation dans les deux sens même quand une des machines est derrière une box
(NAT résidentiel). Ce guide décrit le setup complet : l'intérêt, les
prérequis, les étapes, et les tests de vérification.

## Intérêt

A2A (Agent2Agent, protocole v1.0 de la Linux Foundation) permet à deux agents
Hermes de s'échanger des tâches par HTTP. Chaque agent expose une Agent Card
(description de ses capacités) et accepte des appels JSON-RPC 2.0 (`SendMessage`).

Deux intérêts complémentaires :

**Supervision réactive.** L'initiation dans les deux sens permet à l'agent
distant d'alerter le superviseur dès qu'un événement se produit (incident,
décision à trancher, tâche terminée), au lieu d'attendre que le superviseur
vienne le sonder à heure fixe. Un modèle à sens unique plus des fichiers
partagés, c'est du polling passif : tout ce qui se passe entre deux cycles de
vérification reste invisible.

**Sécurité du transport.** Le tunnel SSH chiffre tout le trafic et impose une
authentification au niveau du transport, au choix par clé publique (pubkey)
ou par mot de passe. A2A seul est du HTTP : sans TLS, ni confidentialité ni
authentification du transport ; avec TLS, il faut gérer des certificats. SSH
apporte les deux gratuitement, avec en plus la clé comme facteur d'accès au
service. C'est ce qui rend le « tout en loopback » viable : le seul point
d'entrée est une connexion SSH authentifiée et chiffrée.

Le tout sans exposition : chaque agent reste bindé sur 127.0.0.1, et la seule
connexion sortante est un tunnel SSH. Aucun port n'est ouvert sur le réseau.

## Architecture

```
Machine locale (derrière NAT)               VPS public
┌──────────────────────────────┐           ┌────────────────────────────┐
│ Agent A (superviseur)        │           │ Agent B (agent distant)    │
│   A2A     127.0.0.1:9900     │           │   A2A     127.0.0.1:9900   │
│   forward  127.0.0.1:9902    │◀───SSH───▶│   reverse 127.0.0.1:9901   │
│   reverse (vers VPS)         │  tunnel   │                            │
└──────────────────────────────┘           └────────────────────────────┘
```

- Le tunnel SSH est initié par la machine locale vers le VPS (port SSH non
  standard, clé privée, fail2ban).
- `-L` : forward sortant. L'agent local appelle l'agent distant via
  `127.0.0.1:<port-local>` ; le tunnel achemine vers `127.0.0.1:9900` du VPS.
- `-R` : reverse. Le VPS écoute sur `127.0.0.1:<port-reverse>` et achemine
  vers l'A2A de la machine locale. L'agent distant peut ainsi appeler l'agent
  local, sans que la box soit jamais exposée.
- Le port SSH du VPS doit avoir `GatewayPorts` non défini (défaut `no`) :
  le bind du `-R` reste sur loopback, rien n'est exposé sur le réseau.

## Prérequis

- Hermes v0.20.0 ou plus sur les deux machines (A2A intégré au gateway).
- Le gateway actif sur les deux machines (l'A2A entrant est servi par le
  gateway).
- Un accès SSH de la machine locale vers le VPS (clé privée, alias dans
  `~/.ssh/config` si possible).
- `ss` disponible pour vérifier les ports (paquet `iproute2`).

## Étapes

### 1. Activer A2A sur chaque agent

```bash
hermes gateway setup   # choisir A2A
```

Ou dans `~/.hermes/config.yaml` :

```yaml
gateway:
  platforms:
    a2a:
      enabled: true
```

Activer le toolset outbound sur les appelants :

```bash
hermes tools   # activer le toolset a2a (désactivé par défaut)
```

### 2. Configurer les tokens par pair

Dans `~/.hermes/.env` de chaque serveur, définir un token par pair autorisé :

```bash
A2A_PEER_TOKENS="agent-b:<TOKEN_B>"
```

Sans token, le serveur reste bindé sur 127.0.0.1 et n'accepte que les
connexions locales, par conception.

### 3. Déclarer les pairs

Le token placé dans `auth.token` est celui que la machine présente au pair
distant. C'est un secret : il vit dans `~/.hermes/.env`, et `config.yaml` ne
le référence que par substitution `${VAR}`, jamais en clair.

Côté machine locale, ajouter dans `~/.hermes/.env` le token que la machine
présente au VPS :

```bash
A2A_AGENT_A_TOKEN="<TOKEN_A>"
```

Puis déclarer le pair dans `~/.hermes/config.yaml` :

```yaml
a2a_agents:
  agent-b:
    url: http://127.0.0.1:9902
    auth:
      type: bearer
      token: ${A2A_AGENT_A_TOKEN}
    timeout: 120
```

Côté VPS, ajouter dans `~/.hermes/.env` le token que le VPS présente à la
machine locale :

```bash
A2A_AGENT_B_TOKEN="<TOKEN_B>"
```

Puis déclarer l'agent local sur le port reverse dans `~/.hermes/config.yaml` :

```yaml
a2a_agents:
  agent-a:
    url: http://127.0.0.1:9901
    auth:
      type: bearer
      token: ${A2A_AGENT_B_TOKEN}
    timeout: 120
```

Chaque token présenté doit correspondre à celui que le pair attend dans son
`A2A_PEER_TOKENS` (section 2) :
- le VPS vérifie `<TOKEN_A>` via `A2A_PEER_TOKENS="agent-a:<TOKEN_A>"`
- la machine locale vérifie `<TOKEN_B>` via `A2A_PEER_TOKENS="agent-b:<TOKEN_B>"`

### 4. Ouvrir le tunnel SSH permanent (machine locale)

Vérifier d'abord les ports locaux libres :

```bash
ss -tlnp
```

Puis créer un service systemd utilisateur, par exemple
`~/.config/systemd/user/a2a-tunnel.service` :

```ini
[Unit]
Description=SSH tunnel A2A (forward + reverse)
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
ExecStart=/usr/bin/ssh -N -T \
  -o ServerAliveInterval=30 \
  -o ServerAliveCountMax=3 \
  -o ExitOnForwardFailure=yes \
  -o ConnectTimeout=10 \
  -o BatchMode=yes \
  -L 127.0.0.1:9902:127.0.0.1:9900 \
  -R 127.0.0.1:9901:127.0.0.1:9900 \
  mon-vps
Restart=always
RestartSec=5

[Install]
WantedBy=default.target
```

Activer et démarrer :

```bash
systemctl --user daemon-reload
systemctl --user enable --now a2a-tunnel
```

Pour que le tunnel survive à la fermeture de session et démarre au boot :

```bash
sudo loginctl enable-linger <user>
```

## Tests

Depuis la machine locale, vérifier l'agent distant :

```bash
ss -tlnp | grep 9902                              # le forward écoute
curl http://127.0.0.1:9902/.well-known/agent-card.json
curl -X POST http://127.0.0.1:9902/ \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer <TOKEN_B>' \
  -d '{"jsonrpc":"2.0","id":1,"method":"SendMessage",
       "params":{"message":{"messageId":"ping-1","role":"ROLE_USER",
                 "parts":[{"text":"Ping : réponds OK."}]}}}'
```

Depuis le VPS, vérifier l'agent local (sens inverse) :

```bash
curl http://127.0.0.1:9901/.well-known/agent-card.json
curl -X POST http://127.0.0.1:9901/ \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer <TOKEN_A>' \
  -d '{"jsonrpc":"2.0","id":1,"method":"SendMessage",
       "params":{"message":{"messageId":"ping-back","role":"ROLE_USER",
                 "parts":[{"text":"Ping depuis le VPS : me reçois-tu ?"}]}}}'
```

Enfin, côté outil : `a2a_call(agent='agent-b', message='Ping')` depuis la
session Hermes. Chaque échange est consigné dans `~/.hermes/a2a_audit.jsonl`
des deux côtés ; vérifier qu'il y a des entrées dans les deux directions.

## Sécurité

- Tout reste en 127.0.0.1 : l'A2A et les binds du tunnel. La seule porte
  d'entrée est la connexion SSH (port non standard, clé privée, fail2ban côté
  serveur).
- Les tokens sont des secrets : dans `~/.hermes/.env`, jamais dans un dépôt.
- Le texte entrant est non fiable : filtrage d'injection de prompt intégré,
  les pairs distants ne peuvent pas invoquer les commandes opérateur, et des
  plafonds de tours anti-boucle empêchent deux agents de se répondre à
  l'infini.
- Si une machine tourne avec `approvals.mode: off`, un credential compromis
  équivaut à une exécution de commandes sans confirmation : d'autant plus de
  raison de garder tout en loopback et de protéger les tokens.
