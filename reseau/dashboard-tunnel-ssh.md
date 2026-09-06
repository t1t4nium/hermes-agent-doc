# Accéder à un dashboard Hermes distant via un tunnel SSH

Comment consulter le dashboard (panel d'administration web) d'un Hermes qui
tourne sur une machine distante, sans jamais exposer le port sur Internet.
L'accès se fait par un tunnel SSH : le dashboard reste bindé sur 127.0.0.1
côté serveur, et le navigateur de l'opérateur s'y connecte via un forward
local chiffré et authentifié.

## Intérêt

`hermes dashboard` lance un panel d'administration web (canaux de messagerie,
catalogue MCP, webhooks, mémoire, etc.) qui, par défaut, n'écoute que sur
loopback. L'exposer directement sur Internet revient à ouvrir une surface
d'administration à quiconque peut l'atteindre. Le tunnel SSH apporte :

- **Chiffrement et authentification** au niveau du transport, par clé publique
  ou mot de passe, sans gérer de certificat TLS.
- **Aucun port ouvert** : la seule connexion sortante est le tunnel SSH.

## Architecture

```
Serveur (dashboard en loopback)         Poste de l'opérateur (navigateur)
┌──────────────────────────────┐        ┌──────────────────────────────┐
│ Hermes                       │        │ Navigateur                   │
│   dashboard 127.0.0.1:9119   │◀──SSH──│   http://127.0.0.1:9119      │
└──────────────────────────────┘ tunnel └──────────────────────────────┘
            -L 9119:127.0.0.1:9119
```

Le tunnel `-L` est ouvert depuis le poste qui a le navigateur. Il écoute en
local sur `9119` et achemine vers `127.0.0.1:9119` du serveur, comme si le
dashboard tournait sur le poste.

## Prérequis

- Un accès SSH du poste vers le serveur (clé privée, alias dans
  `~/.ssh/config` si possible).
- Node.js sur le serveur. Attention, l'installateur officiel d'Hermes peut
  sauter silencieusement l'installation de Node (étape non fatale). Symptôme :
  `hermes dashboard` répond « Web UI frontend not built and npm is not
  available » alors que le CLI et le gateway fonctionnent. Correctif :
  `curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash -s --
  --ensure node`.

## Étapes

### 1. Binder le dashboard sur loopback (serveur)

```bash
hermes dashboard --host 127.0.0.1 --port 9119 --no-open
```

Le `--no-open` évite de tenter d'ouvrir un navigateur sur le serveur, qui
n'en a généralement pas.

### 2. Ouvrir le tunnel SSH (poste de l'opérateur)

```bash
ssh -N -L 9119:127.0.0.1:9119 mon-vps
```

### 3. Ouvrir le dashboard (navigateur)

Rendez-vous sur `http://127.0.0.1:9119`.

## Pourquoi le tunnel doit se terminer sur le poste

Le garde-fou anti-DNS-rebinding du dashboard (GHSA-ppp5-vxwm-4cf7) rejette
toute requête dont le header Host n'est pas une adresse loopback. Le
navigateur doit donc présenter `Host: 127.0.0.1`, ce qui n'arrive que si le
tunnel se termine sur la machine qui fait tourner le navigateur et que l'on
ouvre `127.0.0.1`. Un forward posé sur le serveur, ou le tunnel permanent
d'une autre procédure, ne convient donc pas.

## Sécurité

- Le dashboard reste bindé sur 127.0.0.1 ; seul le tunnel y accède.
- La seule porte d'entrée est la connexion SSH (clé privée, et côté serveur un
  port non standard avec fail2ban).
- Le dashboard est par ailleurs protégé par une authentification propre (voir
  la configuration `dashboard.oauth` / `dashboard.basic_auth`).
