# Hermes WebUI

[Hermes WebUI](https://github.com/nesquena/hermes-webui) est une application web légère et au design sombre exécutée dans votre navigateur pour Hermes Agent. Elle offre une expérience identique à celle du CLI — tout ce que vous pouvez faire depuis un terminal, vous pouvez le faire depuis cette interface. Aucune étape de build, aucun framework, aucun bundler ne sont nécessaires. Juste du Python et du JavaScript de base.

## Prérequis

Hermes Agent doit être installé et fonctionnel.

## Setup

Je m'en sers uniquement pour un setup server accessible uniquement depuis le réseau local. **La configuration décrite ci-dessous n'est pas à exposer sur Internet.**

## Installation

```
git clone https://github.com/nesquena/hermes-webui.git
cd hermes-webui
cp -ai .env.example .env
```

Éditer le fichier `.env` :

```
HERMES_WEBUI_HOST=0.0.0.0
HERMES_WEBUI_PASSWORD=<SUPER_SECRET_PASSWORD>
```

Création du fichier service `~/.config/systemd/user/hermes-webui.service` :

```
[Unit]
Description=Hermes WebUI
After=network.target

[Service]
Environment="HERMES_DUMP_REQUESTS=false"
Type=forking
PIDFile=%h/.hermes/webui.pid
WorkingDirectory=%h/hermes-webui
ExecStart=%h/hermes-webui/ctl.sh start
ExecStop=%h/hermes-webui/ctl.sh stop
ExecReload=%h/hermes-webui/ctl.sh restart
Restart=always
RestartSec=10

[Install]
WantedBy=default.target
```

Activer et démarrer le service :

```
systemctl --user enable hermes-webui.service
systemctl --user start hermes-webui.service
```
