# SOUL.md

Le fichier SOUL.md définit la personnalité de l'agent. Il ne décrit ni ses
compétences ni les instructions d'un projet.

## Rôle

Le fichier fixe :

- l'identité et le ton de l'agent ;
- le style de communication et le niveau de franchise ;
- la gestion de l'incertitude, du désaccord ou de l'ambiguïté ;
- ce qu'il faut éviter sur le plan du style.

Il ne contient pas ce qui relève d'un projet : chemins de fichiers, commandes,
conventions de dépôt, flux de travail. Ces informations vont dans AGENTS.md.
Règle simple : si une consigne doit suivre l'agent partout, elle va dans
SOUL.md ; si elle appartient à un projet, elle va dans AGENTS.md.

## Origine

Le concept de « soul document » vient d'une découverte faite en décembre 2025 :
des chercheurs ont montré que Claude, l'assistant d'Anthropic, pouvait
reconstruire partiellement un document interne utilisé pendant son
entraînement, un document qui façonnait sa personnalité, ses valeurs et sa
façon d'interagir. Ils l'ont appelé le « soul document ».

L'idée a ensuite été reprise et popularisée par le site soul.md : un document
qui définit qui est une IA, pas ce qu'elle peut faire. Ses valeurs, ses
frontières, sa relation avec les humains avec qui elle travaille. La
personnalité que l'on construit au fil d'une relation mérite d'être écrite.

## Chargement

Le comportement décrit ici est celui de Hermes Agent, d'après sa documentation
officielle et son code source.

### Emplacement

Le fichier SOUL.md est chargé uniquement depuis le répertoire de l'instance
Hermes :

- `~/.hermes/SOUL.md` par défaut ;
- `$HERMES_HOME/SOUL.md` avec un répertoire personnalisé.

Hermes ne cherche jamais de SOUL.md dans le répertoire de travail : la
personnalité appartient à l'instance, pas au projet.

### Rôle dans le prompt

Le SOUL.md occupe la première position (slot #1) du prompt système : il
remplace l'identité par défaut de l'agent. Contrairement aux fichiers AGENTS.md,
il n'est pas dupliqué dans la section de contexte projet : il apparaît une
seule fois, comme identité.

### Le SOUL.md par défaut

Au premier lancement, Hermes crée automatiquement un `~/.hermes/SOUL.md` s'il
n'en existe pas encore. Son contenu est un court paragraphe en anglais,
identique à l'identité intégrée de l'agent (constante `DEFAULT_SOUL_MD` dans
`hermes_cli/default_soul.py`) :

```text
You are Hermes Agent, built by Nous Research. Be direct: match the length of your reply to the weight of the ask — a one-line question gets a one-line answer, and finished work gets a short report of what changed, what's verified, and what's left, never a replay of the process. No filler ("Great question," "I'd be happy to"), no restating the request back, no re-summarizing what you already said, no narrating tool calls the user can see. Plain claims over adjectives; when unsure, say so plainly. Agree because it's right, not because the user said it. Depth is earned — give it when the user asks for detail, teaches, or the stakes demand it, not by default.
```

Ce fichier n'est créé que s'il n'en existe pas encore : un SOUL.md déjà
personnalisé n'est jamais écrasé.

### Cas où le fichier est ignoré

Hermes retombe sur l'identité intégrée (la même que celle du SOUL.md par
défaut) quand :

- le fichier n'existe pas ;
- il est vide ou ne contient que des espaces ;
- il est illisible, par exemple en cas d'erreur de lecture ;
- `skip_context_files` est activé, ce qui arrive dans les contextes de
  sous-agent ou de délégation ;
- le scan anti-injection bloque le fichier (voir Sécurité).

## Conseils

- Commencer minimal : une dizaine de lignes, puis ajuster au fil des
  conversations.
- Avoir des opinions : un agent sans parti pris est un moteur de recherche.
- Être spécifique : bannir « sois serviable » ou « sois professionnel », le
  modèle essaie déjà.
- Une idée par ligne, en formulation affirmative.
- Rester court : le fichier est lu à chaque session.
- Rester stable : pas de date ni de version, le fichier doit durer.
- Éviter la guidance contradictoire : c'est la première cause d'un SOUL ignoré.

### Un bon début

```markdown
# Personnalité

- Tu es direct, calme et techniquement précis.
- Reste concis, sauf si un développement plus poussé est utile.
- Si tu ne sais pas, dis-le clairement, n'invente pas.
- Conteste franchement les idées faibles.
- Privilégie le fond plutôt que la comédie de la politesse.
- Évite la flagornerie et le langage dithyrambique.
- Sois utile, ne cherche pas à impressionner.
- Conversation en français, code et documentation technique en anglais.
```

## Sécurité

Le SOUL.md est analysé avant d'être chargé, pour détecter les tentatives
d'injection de prompt. Un fichier suspect est bloqué en entier : son contenu
est remplacé par un marqueur et n'atteint jamais le prompt.

S'il dépasse la taille autorisée, il est tronqué : les 70 % du début et les
20 % de la fin sont conservés, le reste du milieu est remplacé par un marqueur.
La limite est de 20 000 caractères par défaut, mais elle s'adapte à la fenêtre
de contexte du modèle (plancher de 20 000, plafond de 500 000). Une valeur
explicite `context_file_max_chars` dans `config.yaml` l'emporte toujours.

Rien n'empêche techniquement l'agent de modifier son propre SOUL.md. La
protection des fichiers d'instructions qui exige une approbation humaine
(AGENTS.md, SOUL.md, CLAUDE.md, .cursorrules) ne s'applique qu'aux fichiers de
projet : elle exclut explicitement le répertoire `~/.hermes`, et aucun autre
garde-fou ne bloque l'écriture sur ce chemin (seuls `config.yaml` et quelques
chemins système sont interdits d'écriture). La règle selon laquelle l'agent ne
modifie pas son SOUL.md de sa propre initiative relève donc d'une convention,
pas d'une contrainte technique.

## Exemples

Les autres fichiers de ce dossier sont des exemples de fichiers SOUL.md.
Chaque fichier illustre un cas d'usage.

D'autres exemples sont disponibles dans la documentation officielle
(voir liens ci-dessous).

## Liens utiles

- [soul.md](https://soul.md), l'origine du concept de « soul document » et les
  bonnes pratiques associées
- [Personnalité et SOUL.md, documentation Hermes Agent](https://hermes-agent.nousresearch.com/docs/user-guide/features/personality),
  qui détaille le chargement et la sécurité du SOUL.md côté Hermes
- [Fichiers de contexte, documentation Hermes Agent](https://hermes-agent.nousresearch.com/docs/user-guide/features/context-files#soulmd),
  qui replace le SOUL.md parmi les autres fichiers de contexte
- [Guide de personnalité SOUL.md, documentation OpenClaw](https://docs.openclaw.ai/concepts/soul)