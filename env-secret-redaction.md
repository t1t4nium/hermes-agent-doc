# Fuite de credentials via l'agent et solution par masquage exact

## Contexte

Un agent IA autonome (Hermes Agent) tourne sur une machine de confiance et
administre des services, des tunnels SSH, de la config OVH, des boîtes mail.
Les credentials de ces services sont stockés dans `~/.hermes/.env`, chargés
au démarrage de chaque session.

Le problème : ces credentials ont fui à plusieurs reprises, à chaque fois
par le même vecteur (**la sortie d'un outil qui entre dans le contexte de
l'agent**), et ce malgré un SOUL.md volontairement blindé et des règles de
deny en dur sur les commandes.

## Le vecteur de fuite

Une valeur de secret ne fuit pas à la publication (Git) : elle fuit à
l'instant où elle **apparaît dans le résultat d'un outil**, qui devient une
partie de la conversation, qui part vers le modèle cloud.

Exemple concret observé : une commande `bash -x script.sh` a activé le mode
trace, qui a dumpé ligne par ligne le résultat d'un `source ~/.hermes/.env`.
Les valeurs réelles des tokens sont sorties en clair dans l'output du
terminal, donc dans le contexte, donc vers le provider.

## Mesures mises en place et pourquoi elles sont insuffisantes

### 1. Règles de deny en dur (`approvals.deny`)

Des globs fnmatch bloquent inconditionnellement les commandes qui touchent
au `.env` (`cat`, `grep`, `source`, etc.), même en mode yolo.

```yaml
# config.yaml
approvals:
  deny:
    - '*cat*.env*'
    - '*less*.env*'
    - '*more*.env*'
    - '*head*.env*'
    - '*tail*.env*'
    - '*grep*.env*'
    - '*sed*.env*'
    - '*awk*.env*'
    - '*cut*.env*'
    - '*base64*.env*'
    - '*strings*.env*'
    - '*od*.env*'
    - '*xxd*.env*'
    - '*diff*.env*'
    - '*cp*.env*'
    - '*scp*.env*'
    - '*rsync*.env*'
    - '*install*.env*'
    - '*cat*.hermes/auth.json*'
    - '*grep*.hermes/auth.json*'
```

**Insuffisant car** : elles couvrent le texte de la commande. Or l'agent
trouve des chemins indirects qui ne contiennent pas le motif `.env`
(une commande `bash -x` sur un script qui fait lui-même le `source`,
une redirection, un lecteur indirect). C'est une garde de surface, pas une
barrière sur la donnée elle-même.

### 2. Redaction native (`security.redact_secrets`)

Hermes masque les sorties d'outils qui **ressemblent** à des credentials
(préfixes connus `sk-`, `ghp_`, `hf_`, assignations `KEY=value` avec
mot-clé secret).

**Insuffisant car** : la redaction est basée sur la *forme*, pas sur les
*valeurs*. Deux trous structurels :

- Elle ne reconnaît que les formes connues. Un token opaque sans préfixe
  reconnu (token Mastodon, secret KuCoin, mot de passe mail) ne matche
  aucune regex.
- La passe d'assignation ENV (`KEY=value`) n'est activée que pour deux
  catégories de commandes : les dumps d'environnement (`env`, `printenv`,
  `set`, `export`) et les lectures directes de `.env` (`cat .env`). Pour
  toute autre commande (dont `bash -x`), la passe est sautée, c'est
  exactement la porte de sortie constatée.

### 3. Lecture `.env` bloquée via l'outil fichier (`file_safety.py`)

`read_file` refuse de lire les `.env`.

**Insuffisant car** : le terminal tourne avec le même user Unix. L'agent
peut toujours lire le fichier via un shell indirect. La doc Hermes le dit
explicitement : ce n'est pas une frontière de sécurité, c'est de la
défense en profondeur.

### 4. SOUL.md blindé (règles secrets répétées)

```
## Style
- Tu respectes les secrets et tu ne cherches pas à les connaître.

## Posture technique
- Pour utiliser un secret (clé API, mot de passe, passphrase), charge-le dans l'environnement sans lire ou afficher sa valeur.

## À éviter
- Lire le contenu d'un fichier de secrets : l'output exposerait le secret dans la conversation.

## À ne PAS faire
- Divulguer des secrets, des données sensibles ou personnelles, dans une réponse, un fichier ou un message.
```

**Insuffisant car** : un modèle est probabiliste. Aucune formulation
textuelle ne garantit 100 % d'obéissance. La répétition a un rendement
décroissant et dilue le reste du fichier.

### 5. Backend Docker / SSH (isolation)

Documenté comme la réponse à un agent *hostile ou compromis*, pas à un
agent honnête mais faillible. Isole l'exécution mais ne corrige pas la
redaction : si une valeur atteint malgré tout le conteneur, la fuite se
reproduit. Et pour un agent qui administre la machine hôte (clés SSH,
services locaux), l'isolation casse l'accès légitime.

## La cause racine

Toutes les mesures agissaient sur **la commande** ou **la forme** de la
donnée. Aucune ne protégeait la **valeur réelle** du secret. Tant que la
valeur d'un secret peut apparaître dans un output, quelle que soit la
commande qui le produit, elle finit dans le contexte.

## La solution : masquage par valeur exacte (plugin)

Un plugin Hermes (`env-secret-redactor`) qui ferme la faille sur la donnée
elle-même, de façon indépendante de la commande et du format :

1. Au chargement, il lit `~/.hermes/.env` avec le parseur officiel
   (`agent.secret_scope.load_env_file`), sans jamais afficher les valeurs.
2. Il ne retient que les variables dont le **nom** matche un mot-clé
   secret (`TOKEN`, `PASSWORD`, `SECRET`, `KEY`, `CREDENTIAL`, `AUTH`, etc.),
   insensible à la casse. Les noms non-secrets (`USER`, `ADDRESS`, `URL`,
   `HOST`, `INSTANCE`, `CHANNEL`, `TIMEOUT`) sont ignorés.
3. Il installe les hooks natifs `transform_terminal_output` et
   `transform_tool_result`, qui remplacent chaque valeur exacte par un
   sentinel inerte (`«redacted-secret»`) avant que l'output n'entre dans
   le contexte.

Deux règles découlent du filtrage par nom :

- **Le nom est toute la frontière.** Une fois qu'un nom matche, la valeur
  est un secret *quel que soit son contenu* (`@`, `://`, n'importe quoi) et
  est toujours masquée. Il n'y a pas d'exception fondée sur la forme de la
  valeur : un mot de passe Redis `redis://:secret@host` ou un DSN avec
  userinfo est masqué au même titre qu'un token opaque.
- **Le motif de nom est surchargeable.** Le matcher interne est le défaut.
  On peut le remplacer par une regex via
  `plugins.env-secret-redactor.secret_name_pattern` dans `config.yaml`
  (comparaison insensible à la casse). Une regex invalide est ignorée avec
  un warning : le défaut sûr reste en place.

### Pourquoi des hooks, pas le registre de redaction

Le registre `register_redaction_patterns` produit un sentinel
`«redacted:VALEUR…»` dont le label est, pour une valeur exacte opaque,
la valeur entière (il n'y a pas de préfixe court à isoler). La donnée
fuirait donc encore sur les surfaces `file_read`. Les hooks font un
masquage **total et sans label** : aucun octet de la valeur ne subsiste.

### Effets de bord maîtrisés

- **Correspondance exacte** : une valeur similaire d'un seul caractère
  ne déclenche rien.
- **Filtrage par nom** : les adresses mail, logins, URLs et IDs stockés
  sous des noms non-secrets ne sont jamais enregistrés, donc une sortie
  légitime contenant une adresse ou un login n'est pas masquée.
- **Ordre de remplacement** : les valeurs sont triées par longueur
  décroissante avant masquage. Si un secret est le préfixe d'un autre
  (ex. `abc` et `abcdef`), le plus long est remplacé en premier ; sans ce
  tri, la correspondance courte casserait la longue et laisserait un
  résidu (`def`) en clair.
- **Non-ingérence entre plugins** : chaque hook renvoie `None` quand rien
  n'a été masqué, au lieu de renvoyer la chaîne inchangée. Le framework
  applique le premier retour chaîne gagnant ; renvoyer toujours une
  chaîne aurait court-circuité les autres plugins branchés sur le même
  hook.

## Validation en conditions réelles

Le scénario qui avait fui (`bash -x` sur un script sourçant le `.env`) a
été rejoué après installation du plugin, sur de vraies clés :

```
++ GITHUB_TOKEN=«redacted-secret»
+ echo GITHUB_TOKEN=«redacted-secret»
```

Chacune des huit valeurs secrètes du `.env` est remplacée par le sentinel
inerte, tandis que les non-secrets (adresse mail, hôte IMAP, identifiant
d'utilisateur, instance, URL) restent en clair : le filtrage par nom ne
produit pas de faux positif.

Le plugin est couvert par 22 tests hermétiques (`HERMES_HOME` redirigé vers
un répertoire temporaire, `.env` synthétique) : le vrai `~/.hermes/.env`
n'est jamais touché.

## Déploiement

Le code vit dans un dépôt git séparé du répertoire d'installation. Le
plugin s'installe par la commande officielle Hermes, qui clone depuis Git
et gère l'activation :

```bash
hermes plugins install t1t4nium/hermes-agent-env-secret-redactor
```

Le dépôt est publié et le plugin mis à jour par ce biais.

## Leçon

Pour des secrets sur une machine administrée par un agent, la défense
efficace protège la **valeur**, pas la commande ni la forme. Un agent
honnête mais faillible doit être incapable de voir la valeur dans un
output, quelle que soit la manière dont elle y arrive.