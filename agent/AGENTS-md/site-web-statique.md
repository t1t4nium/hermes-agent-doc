# Site web d'une photographe

Dans cet exemple, l'agent est en charge de mettre à jour et publier un site web
construit avec un SSG (Static Site Generator), et d'aider l'utilisateur à
rédiger les pages.

Voici à quoi pourrait ressembler le fichier AGENTS.md déposé à la racine du
dossier du projet.

```markdown
# Site web de Jane Doe

Ce dossier contient tous les fichiers du site web de Jane Doe, photographe
indépendante spécialisée dans le portrait et les mariages. Le site présente son
portfolio et permet aux visiteurs de la contacter. Les pages sont en français.

## Informations générales

- URL du site : https://www.jane-photo.example
- E-mail de contact : janephoto@example.com
- Instagram : https://www.instagram.com/janephotoexample/

## Stack et commandes

Site statique construit avec le SSG [Jekyll](https://jekyllrb.com/docs/) en suivant
les conventions et recommandations de la documentation officielle.

### Commandes Jekyll

- Construire le site : `bundle exec jekyll build`
- Servir le site en local : `bundle exec jekyll serve`

### Hébergement web

L'hébergement est assuré par OVH :
- Serveur SFTP : `ftp.example.hosting.ovh.net`
- Identifiant et clé : déjà en place dans la configuration SSH
- Répertoire de déploiement distant : `www`
- Commande de connexion : `sftp ftp.example.hosting.ovh.net`

### Déploiement du site

1. Construire le site
2. Supprimer le contenu du dossier distant `www` sur l'hébergement web
3. Pousser le contenu du répertoire `_site` vers le répertoire distant `www`
4. Vérifier que la nouvelle version est en ligne

## Structure

Configuration du site : `_config.yml`

Le portfolio est divisé en deux galeries :
- Les mariages `portfolio/mariages.md`
- Les portraits `portfolio/portraits.md`

Organisation des photos :
- Mariages : `assets/photos/mariages/`
- Portraits : `assets/photos/portraits/`

Les autres pages :
- Accueil : `index.md`
- Prestations : `prestations.md`
- Contact : `contact.md`
- Mentions légales : `mentions-legales.md`

Le dossier `_site` est généré par la commande de construction (build) du site.
Il ne faut donc pas le modifier manuellement.

## Règles et conventions

- Pour la rédaction : ton chaleureux mais sobre, sans jargon technique. Pas d'émoji décoratif.
- Les photos sont livrées prêtes à publier, il ne faut donc pas les modifier.
- Le contenu peut être modifié par l'agent et par l'utilisateur.
- L'agent doit respecter les modifications de l'utilisateur et ne pas chercher à les revert.
- Toute modification doit être versionnée par l'agent avec Git.
- Demander l'accord de l'utilisateur avant déploiement.
```
