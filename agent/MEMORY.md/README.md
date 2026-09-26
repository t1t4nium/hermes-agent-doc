# MEMORY.md

Le fichier MEMORY.md est l'un des deux fichiers qui forment la mémoire
persistante de l'agent. Il contient les notes personnelles de l'agent : faits
d'environnement, conventions, leçons apprises. L'autre fichier, USER.md, décrit
le profil de l'utilisateur.

Un raccourci pour distinguer les quatre fichiers qui façonnent l'agent :

- SOUL.md : qui est l'agent ;
- USER.md : qui est l'utilisateur ;
- MEMORY.md : ce que l'agent a appris ;
- AGENTS.md : ce dont le projet a besoin.

## Rôle

Le fichier MEMORY.md contient les notes personnelles de l'agent, ce qu'il
doit retenir sur l'environnement, les flux de travail et les leçons apprises,
d'une session à l'autre :

- les faits d'environnement (système, outils, structure d'un projet) ;
- les conventions et la configuration d'un projet ;
- les particularités d'outils et les contournements découverts ;
- le journal du travail terminé ;
- les compétences et techniques qui ont fonctionné.

Il ne décrit pas l'utilisateur : ces informations vont dans USER.md.

## Emplacement

- `~/.hermes/memories/MEMORY.md` par défaut ;
- `$HERMES_HOME/memories/MEMORY.md` avec un répertoire personnalisé ;
- `~/.hermes/profiles/<nom>/memories/MEMORY.md` pour un profil.

La mémoire est propre à chaque profil. Deux profils ne partagent pas leurs
notes.

## Qui l'écrit

L'agent, via l'outil `memory` avec la cible `memory`. Il peut ajouter (`add`),
remplacer (`replace`) ou supprimer (`remove`) une entrée. Il n'y a pas d'action
de lecture : le contenu est injecté dans le prompt système au démarrage de la
session.

Une précision qui évite bien des malentendus : la mémoire ne persiste que si le
modèle appelle réellement l'outil `memory`. Une phrase du type « c'est noté »
sans appel d'outil n'écrit rien sur disque.

## Fonctionnement

### Instantané figé

Au démarrage de chaque session, les entrées sont lues sur disque et rendues dans
le prompt système sous la forme d'un bloc figé. L'en-tête indique le magasin, le
pourcentage d'occupation et le nombre de caractères ; les entrées sont séparées
par le signe §.

L'injection est capturée une fois au démarrage et ne change plus en cours de
session, pour préserver le cache de préfixe du modèle. Quand l'agent ajoute,
remplace ou supprime une entrée en cours de session, le changement est écrit sur
disque immédiatement, mais il n'apparaît dans le prompt système qu'à la session
suivante. Les réponses de l'outil montrent toujours l'état à jour.

Concrètement : une information mémorisée pendant une session est visible dans la
session suivante, pas dans une session déjà en cours. Pour en profiter, il faut
clôturer la session (`/new`) puis en ouvrir une nouvelle.

### Limite de capacité

La mémoire a une limite stricte de 2 200 caractères (environ 800 tokens). Il n'y
a pas de compaction automatique : si un ajout ferait dépasser la limite, l'outil
renvoie une erreur au lieu de supprimer des entrées silencieusement. L'agent
fait alors de la place lui-même, dans le même tour, en consolidant ou en
supprimant des entrées avant de réessayer.

### Avec et sans approbation

Par défaut (`memory.write_approval: false`), l'agent écrit librement, y compris
depuis la revue d'auto-amélioration qui tourne en arrière-plan après un tour.

Avec `memory.write_approval: true`, toute écriture demande une approbation :

- dans le CLI interactif, les écritures en avant-plan déclenchent une demande en
  ligne, directement dans la conversation ;
- partout ailleurs (plateformes de messagerie, scripts, revue en arrière-plan),
  les écritures sont mises en attente puis examinées avec `/memory pending`,
  `/memory approve` et `/memory reject`.

Une nuance : les remplacements et suppressions issus de la revue en arrière-plan
sont mis en attente même quand l'approbation est désactivée.

## Ce qu'on y met

À mémoriser de façon proactive :

- les préférences techniques de l'utilisateur (« je préfère TypeScript à
  JavaScript ») ;
- les faits d'environnement (« ce serveur tourne sous Debian 12 avec
  PostgreSQL 16 ») ;
- les corrections (« ne pas utiliser sudo pour Docker, l'utilisateur est dans le
  groupe docker ») ;
- les conventions (« tabulations, largeur 120 caractères, docstrings style
  Google ») ;
- le travail terminé (« base migrée de MySQL vers PostgreSQL le 2026-01-15 ») ;
- les demandes explicites (« la rotation de ma clé API est mensuelle »).

À éviter :

- les informations triviales ou évidentes ;
- les faits facilement redécouvrables par une recherche ;
- les dumps de données brutes (blocs de code, journaux, tableaux) ;
- l'éphémère de session (chemins temporaires, contexte de débogage ponctuel) ;
- ce qui est déjà dans SOUL.md ou AGENTS.md.

Pour un emplacement dont l'agent a besoin à chaque exécution d'une tâche
récurrente, une skill est souvent un meilleur endroit qu'une entrée de mémoire :
elle ne se charge que quand elle est pertinente et ne consomme pas le budget de
2 200 caractères.

### Exemples d'entrées

```text
Le serveur de staging est joignable sur le port SSH 2222, pas le 22. Clé dans ~/.ssh/staging_ed25519.
§
Le projet ~/code/api utilise Go 1.22, sqlc pour les requêtes SQL et le routeur chi. Tests avec make test.
§
L'utilisateur est dans le groupe docker : ne pas utiliser sudo pour les commandes Docker.
```

## Conseils

- Privilégier des entrées compactes et denses en information, qui regroupent
  plusieurs faits liés.
- Écrire des faits déclaratifs, pas des consignes : « l'utilisateur préfère des
  réponses concises », pas « réponds toujours de façon concise ».
- Consolider avant d'ajouter quand la mémoire dépasse 80 % de sa capacité.
- Un fait qui perd sa pertinence en moins d'une semaine appartient à l'historique
  de session, pas à la mémoire.

## Sécurité

Les entrées mémoire sont scannées avant d'être acceptées, car elles sont
injectées dans le prompt système. Un contenu qui correspond à des motifs de
menace (injection de prompt, exfiltration d'identifiants, portes dérobées SSH)
ou qui contient des caractères Unicode invisibles est bloqué.

Les doublons exacts sont rejetés automatiquement.

## Mon usage

La mémoire est activée et les écritures sont soumises à mon approbation
(`memory.write_approval: true`). Je relis les écritures en attente avec
`/memory pending`, puis `/memory approve` ou `/memory reject`.

Ma règle de tri : uniquement du transversal.

- Un fait d'environnement global, une préférence durable, une leçon qui vaut
  dans toutes les sessions : oui.
- La structure, une convention ou une configuration d'un projet précis : non,
  elles vont dans l'AGENTS.md de ce projet.
- Une procédure réutilisable : non, elle va dans une skill.

La mémoire est chargée en entier dans chaque session. Une note de projet y
polluerait le contexte de toutes les autres sessions, même sans rapport avec ce
projet. J'audite la mémoire avec `/journey`, qui liste, édite et supprime ce que
l'agent a appris.

## Liens utiles

- [Mémoire persistante, documentation Hermes Agent](https://hermes-agent.nousresearch.com/docs/user-guide/features/memory),
  qui détaille le fonctionnement, la capacité et l'approbation des écritures
- [Quel fichier fait quoi, documentation Hermes Agent](https://hermes-agent.nousresearch.com/docs/user-guide/which-file-does-what),
  qui replace MEMORY.md parmi SOUL.md, USER.md et AGENTS.md
