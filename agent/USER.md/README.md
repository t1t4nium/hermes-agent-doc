# USER.md

Le fichier USER.md est l'un des deux fichiers qui forment la mémoire persistante
de l'agent. Il contient le profil de l'utilisateur : son identité, ses
préférences, son style de communication. L'autre fichier, MEMORY.md, contient
les notes personnelles de l'agent.

Un raccourci pour distinguer les quatre fichiers qui façonnent l'agent :

- SOUL.md : qui est l'agent ;
- USER.md : qui est l'utilisateur ;
- MEMORY.md : ce que l'agent a appris ;
- AGENTS.md : ce dont le projet a besoin.

## Rôle

Le fichier USER.md contient ce que l'agent doit savoir sur la personne avec qui
il travaille :

- le nom, le rôle et le fuseau horaire ;
- les préférences de communication (concis ou détaillé, formats) ;
- les irritants et ce qu'il faut éviter ;
- les habitudes de travail ;
- le niveau technique.

Il ne décrit ni la personnalité de l'agent (SOUL.md), ni les instructions d'un
projet (AGENTS.md), ni les connaissances de l'agent (MEMORY.md).

## Emplacement

- `~/.hermes/memories/USER.md` par défaut ;
- `$HERMES_HOME/memories/USER.md` avec un répertoire personnalisé ;
- `~/.hermes/profiles/<nom>/memories/USER.md` pour un profil.

Le profil utilisateur est propre à chaque profil Hermes. Deux profils ne partagent pas le
même USER.md.

## Qui l'écrit

L'agent, via l'outil `memory` avec la cible `user`. Il peut ajouter (`add`),
remplacer (`replace`) ou supprimer (`remove`) une entrée. Il n'y a pas d'action
de lecture : le contenu est injecté dans le prompt système au démarrage de la
session.

Il faut bien distinguer SOUL.md et USER.md. SOUL.md est un fichier de
personnalité que l'utilisateur édite directement, tandis que USER.md fait partie
de la mémoire persistante et est écrit par l'agent. Vouloir mettre des
informations sur soi dans SOUL.md ne remplit pas USER.md : pour qu'un fait
figure dans le profil, il faut le dire à l'agent, qui l'enregistre.

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

### Limite de capacité

Le profil a une limite stricte de 1 375 caractères (environ 500 tokens). Il n'y
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

C'est la réponse au cas « l'agent a retenu une fausse hypothèse sur moi » :
activer l'approbation fait que chaque écriture, y compris celles de la revue en
arrière-plan, attend un oui ou un non avant d'entrer dans le profil.

## Ce qu'on y met

À mémoriser de façon proactive :

- le nom, le rôle et le fuseau horaire ;
- les préférences de communication (concis ou détaillé, formats) ;
- les irritants et ce qu'il faut éviter ;
- les habitudes de travail ;
- le niveau technique.

À éviter :

- les informations triviales ou évidentes ;
- les faits facilement redécouvrables par une recherche ;
- les dumps de données brutes ;
- l'éphémère de session ;
- ce qui est déjà dans SOUL.md ou AGENTS.md.

### Exemples d'entrées

```text
Préfère les réponses en français et les explications concises.
§
Fuseau horaire Europe/Paris, travaille surtout le matin.
§
Éviter le jargon inutile, aller droit au but.
```

## Conseils

- Privilégier des entrées compactes et denses en information.
- Écrire des faits déclaratifs, pas des consignes : « l'utilisateur préfère des
  réponses concises », pas « réponds toujours de façon concise ».
- Consolider avant d'ajouter quand le profil dépasse 80 % de sa capacité.

## Sécurité

Les entrées du profil sont scannées avant d'être acceptées, car elles sont
injectées dans le prompt système. Un contenu qui correspond à des motifs de
menace (injection de prompt, exfiltration d'identifiants, portes dérobées SSH)
ou qui contient des caractères Unicode invisibles est bloqué.

Les doublons exacts sont rejetés automatiquement.

## Mon usage

Le profil est activé et les écritures sont soumises à mon approbation, comme la
mémoire (`memory.write_approval: true`).

Le profil porte uniquement ce qui me décrit de façon durable : identité,
préférences, style de communication, habitudes. Rien de technique, rien de scopé
à un projet. Par nature, c'est du transversal.

Quand un fait sur moi est faux ou périmé, je le corrige en le disant à l'agent,
qui remplace l'entrée.

## Liens utiles

- [Mémoire persistante, documentation Hermes Agent](https://hermes-agent.nousresearch.com/docs/user-guide/features/memory),
  qui détaille le fonctionnement, la capacité et l'approbation des écritures
- [Quel fichier fait quoi, documentation Hermes Agent](https://hermes-agent.nousresearch.com/docs/user-guide/which-file-does-what),
  qui replace USER.md parmi SOUL.md, MEMORY.md et AGENTS.md
