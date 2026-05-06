# Première installation

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
- **discard** active le TRIM en ligne pour maintenir les performances SSD sur la durée

```
UUID=UUID_PARTITION / ext4 errors=remount-ro,noatime,discard 0 1
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

## Pour finir

J'ai écrit le fichier `SOUL.md` pour définir le ton, la personnalité et les
informations de base à toutes les sessions.

Il ne faut pas oublier de redémarrer la gateway après modification du modèle ou
du fichier `SOUL.md`.
