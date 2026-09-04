# AGENTS.md

Un fichier AGENTS.md est un fichier de contexte. Son rôle est de donner à un
agent IA les informations nécessaires pour travailler sur un projet ou dans un
espace de travail.

## Rôle

Le fichier indique à l'agent :
- comment le projet est structuré ;
- quelles conventions suivre ;
- quelles instructions particulières respecter ;
- toute information utile ou nécessaire au projet.

## Chargement

Le comportement décrit ici est celui de Hermes Agent, d'après sa documentation
officielle. Le principe général vaut pour la plupart des agents qui lisent un
fichier AGENTS.md.

### Priorité des fichiers de contexte

À l'ouverture d'une session, un seul type de fichier de contexte projet est
chargé, selon la priorité (le premier trouvé l'emporte) :

`.hermes.md` > `AGENTS.override.md` > `AGENTS.md` > `CLAUDE.md` > `.cursorrules`

Le fichier SOUL.md est chargé indépendamment, comme identité de l'agent.

Le fichier `AGENTS.override.md` permet d'avoir une version personnelle,
généralement ignorée par Git, qui remplace le fichier AGENTS.md versionné sans
le modifier.

### Chaîne de répertoires

Quand le répertoire de travail se trouve dans un dépôt Git, Hermes charge une
chaîne fusionnée de fichiers AGENTS.md : celui de la racine du dépôt d'abord,
puis ceux des répertoires intermédiaires jusqu'au répertoire de travail. Les
fichiers plus profonds arrivent plus tard dans le contexte et priment donc sur
les plus généraux.

En dehors d'un dépôt Git, seul le répertoire de travail est vérifié, jamais ses
parents. Un AGENTS.md placé dans un répertoire personnel ne peut donc pas fuiter
dans une session sans rapport.

### Découverte progressive

Au démarrage, Hermes charge le AGENTS.md du répertoire de travail dans le prompt
système. Quand l'agent navigue dans des sous-répertoires pendant la session,
Hermes découvre les fichiers de contexte qui s'y trouvent et les injecte au
moment où ils deviennent pertinents. Cela évite de gonfler le prompt système et
préserve le cache de prompt.

## Ce qu'on y met

- la structure du projet ;
- les conventions à suivre ;
- les instructions spécifiques au projet ;
- les chemins et ports utiles ;
- ce qu'il ne faut pas faire.

## Conseils

- Rester concis : l'agent lit le fichier à chaque tour.
- Structurer avec des titres de niveau 2.
- Donner des exemples concrets de code, de formes d'API et de nommage.
- Mentionner ce qu'il ne faut pas faire.
- Lister les chemins et ports clés.
- Mettre le fichier à jour au fil du projet : un contexte périmé vaut pire que
  pas de contexte.

## Sécurité

Les fichiers de contexte sont analysés avant d'être chargés, pour détecter les
tentatives d'injection de prompt. Un fichier suspect est bloqué. Ce garde-fou ne
remplace pas une relecture des fichiers AGENTS.md dans un dépôt partagé que vous
n'avez pas écrit vous-même.

## Exemples

Les autres fichiers de ce dossier sont des exemples de fichiers AGENTS.md.
Chaque fichier illustre un cas d'usage.

## Liens utiles

- [agents.md](https://agents.md), la présentation du format AGENTS.md, de son
  adoption par les agents de codage et les bonnes pratiques associées
- [Fichiers de contexte, documentation Hermes Agent](https://hermes-agent.nousresearch.com/docs/user-guide/features/context-files#agentsmd),
  qui détaille le chargement et la sécurité des fichiers AGENTS.md côté Hermes
