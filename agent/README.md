# Agent

Ce dossier documente tout ce qui façonne un agent Hermes. Chaque sous-dossier décrit
le rôle du fichier de contexte ou d'un concept clé, le fonctionnement d'après la
documentation officielle, et mes règles d'usage.

## Sommaire

- Les fichiers de contexte : SOUL.md, AGENTS.md, MEMORY.md, USER.md.

Ce sommaire s'étoffera avec les skills et les autres concepts clés au fil des
ajouts.

## Les fichiers de contexte

Quatre fichiers façonnent l'agent. Chacun a un rôle, un auteur et un emplacement
précis :

| Fichier | Rôle | Qui l'écrit | Emplacement |
| --- | --- | --- | --- |
| SOUL.md | Qui est l'agent | L'utilisateur | `~/.hermes/SOUL.md` |
| AGENTS.md | Ce dont le projet a besoin | L'utilisateur | Racine du projet |
| MEMORY.md | Ce que l'agent a appris | L'agent | `~/.hermes/memories/MEMORY.md` |
| USER.md | Qui est l'utilisateur | L'agent | `~/.hermes/memories/USER.md` |

Chacun est décrit dans son sous-dossier : SOUL.md, AGENTS-md, MEMORY.md, USER.md.

## Règle de tri

Chaque information a un domicile :

- une consigne qui suit l'agent partout (ton, style, franchise) va dans
  SOUL.md ;
- une structure, une convention ou une configuration propre à un projet va dans
  l'AGENTS.md de ce projet ;
- un fait transversal appris par l'agent (environnement global, préférences
  durables, leçons) va dans MEMORY.md ;
- une information sur l'utilisateur (identité, préférences, style) va dans
  USER.md ;
- une procédure réutilisable (savoir-faire, commandes, pièges) va dans une
  skill.

La frontière qui compte : ce qui relève d'un projet précis ne va jamais dans
MEMORY.md. La mémoire est chargée en entier dans chaque session ; une note de
projet y polluerait le contexte de toutes les autres sessions, même sans rapport
avec ce projet.

## Contrôle des écritures

- Mémoire (MEMORY.md et USER.md) : écriture soumise à approbation
  (`memory.write_approval: true`).
- Skills : écriture libre (`skills.write_approval: false`).
- AGENTS.md : fichier d'instructions d'un projet, modifié par l'agent seulement
  avec approbation.
- SOUL.md : édité par l'utilisateur ; l'agent ne le modifie pas de sa propre
  initiative (convention).

Les écritures de mémoire en attente se relisent avec `/memory pending`, puis
`/memory approve` ou `/memory reject`.
