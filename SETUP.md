# Setup

Je mets ici les commandes et instructions utiles pour faciliter les setup Hermes Agent sur Debian GNU/Linux 13 x86_64.

## Common

Instructions communes à tout setup.

### Optimisation

Ajuster le swappiness pour que le système utilise d'avantage la RAM et swap moins souvent :

```
sysctl vm.swappiness=10
echo vm.swappiness=10 >> /etc/sysctl.d/25-custom.conf
```

### Prérequis

Installation des paquets requis par Hermes Agent :

```
apt install build-essential curl ffmpeg git imagemagick libffi-dev openjdk-25-jre-headless python3-dev ripgrep
```

## Server

Instructions uniquement pour setup type serveur, donc sur une machine allumée 24/7.

### Optimisation

Pour améliorer les performances du disque, ajouter `noatime` pour désactiver l'access time sur la partition principale dans `/etc/fstab` :

```
UUID=<PART_UUID> / ext4 errors=remount-ro,noatime 0 1
```
