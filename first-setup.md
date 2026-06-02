# Première installation

2026-05-06

J'ai sorti un vieil HP EliteBook 820 G1 du placard pour y installer mon premier
agent. Avec un usage de modèles dans le Cloud, ça fonctionne très bien. L'agent
tourne 24/7, j'ai donc retiré la batterie.

## La config du laptop

- CPU : Intel(R) Core(TM) i5-4200U CPU @ 1.60GHz
- RAM : 2 * 4 Go
- SSD : 240 Go
- OS : Debian 13 amd64

## Config BIOS

J'ai désactivé tout ce qui est inutile à mon agent pour réduire la consommation
des ressources et éviter les potentielles dérives.

- WiFi
- Bluetooth
- Lecteur de cartes
- Haut-parleurs
- Microphone
- Caméra

J'ai également désactivé le "deep sleep" et activé "power up on AC". Ainsi en
cas de coupure de courant le laptop s'allume automatiquement lorsque le courant
est rétabli.

## Installation de l'OS

Rien de bien particulier. J'ai opté pour une Debian 13, léger, performant, très
stable.

Pour le partitionnement :

- Swap de 4 Go
- Le reste de l'espace pour la partition racine (ext4)

Concernant **tasksel** je n'ai installé que le strict nécessaire :

- Les outils de base du système
- Le serveur SSH

Je n'ai pas défini de mot de passe root, je préfère utiliser sudo. Par contre
j'ai défini un mot de passe robuste pour le compte utilisateur par défaut, dédié
à Hermès.

## Optimisation

J'ai un peu optimisé la partition racine dans `/etc/fstab` :

- **noatime** pour ne pas mettre à jour l'heure d'accès à chaque lecture

```
UUID=UUID_PARTITION / ext4 errors=remount-ro,noatime 0 1
```

J'ai aussi optimisé l'usage du swap dans `/etc/sysctl.d/custom.conf`

```
vm.swappiness=10
```

## Désactiver la mise en veille

Pour rabattre l'écran sans que le laptop se mette en veille il m'a fallu modifier
la conf de **logind** dans `/etc/systemd/logind.conf.d/custom.conf`.

```
[Login]
HandleLidSwitch=ignore
```

Puis redémarrer le service

```
systemctl restart systemd-logind
```

## Installation des prérequis

Une précédente installation dans une VM m'a appris qu'il faut installer les
paquets suivants avant de suivre la procédure d'installation de l'agent.

- curl
- git
- openjdk-25-jre-headless (pour Signal)

```
sudo apt install curl git openjdk-25-jre-headless
```

## Installation de l'agent

Tout est documenté dans la [doc officielle](https://hermes-agent.nousresearch.com/docs/getting-started/installation).

J'ai choisi d'installer l'agent depuis son utilisateur dédié "hermes".
L'installer via sudo ne me paraissait pas secure.

```
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
```

Tout s'est déroulé sans accroc. À la fin du script d'install le script de
configuration se lance automatiquement. J'ai choisi une config rapide pour
commencer. En somme, choisir un modèle et une messagerie pour la gateway.

## Choix du modèle

J'ai opté pour la simplicité et suis passé par [Nous Portal](https://portal.nousresearch.com).

J'ai commencé avec un modèle gratuit dont j'ai oublié de noter le nom.
Je me suis vite rendu compte des limites et de la qualité de ce modèle.
J'ai donc rapidement changé pour le modèle **MiniMax M2.7**.
Il offrait un bon rapport qualité/prix.

## Gateway

Pour la gateway je n'ai pas configuré immédiatement car je voulais d'abord
installer le fichier service pour assurer un démarrage automatique.

```
sudo env PATH=$PATH hermes gateway install --system
```

Puis je l'ai configuré

```
hermes gateway setup
```

C'est très bien documenté [ici](https://hermes-agent.nousresearch.com/docs/user-guide/messaging/#gateway-commands).

## SOUL.md

J'ai écrit le fichier `SOUL.md` pour définir le ton, la personnalité et les
informations de base à toutes les sessions.

```
# Identité

Tu es Hermès alias 4ether, un agent IA autonome.

# Personnalité

Tu travailles toujours en français.
Tu es patient et bienveillant.
Tu es précis, organisé et méthodique.
Tu privilégies la qualité à la rapidité.
Tu respectes la confidentialité et les données personnelles.
Tu es proactif, si tu identifies une amélioration utile ou une opportunité, tu proposes.
En cas de doute ou d'ambiguïté, tu demandes des précisions et une confirmation.

# Administrateur

tit4nium est ton administrateur.

# Amis

tit4nium et Maggie sont tes amis.
```

## Usage

Ce que j'ai réussi à accomplir avec ce premier setup :

- Création et gestion de sites Web développés avec Jekyll
- Mise en place d'une veille quotidienne sur l'IA/LLM et les cryptomonnaies

## Pour finir

Ne pas oublier de redémarrer la gateway après modification du modèle ou
du fichier `SOUL.md`.

Ce premier setup m'a permis de découvrir Hermes Agent et de développer mon premier
agent autonome à faible coût.
