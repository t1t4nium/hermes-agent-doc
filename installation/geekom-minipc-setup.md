# Installation sur un mini PC

Pour des raisons de sécurité et de confidentialité, l'usage
d'un modèle local et la création d'un agent par utilisateur me
paraissait de plus en plus évident.

Pour cette seconde installation, je me suis offert un **GEEKOM A5 Mini PC**.
Je m'étais naïvement fixé l'objectif de pouvoir charger le modèle
[Qwen 2.5 72B Instruct GGUF](https://huggingface.co/bartowski/Qwen2.5-72B-Instruct-GGUF)
en quantisation Q4\_K\_M, qui devait tenir dans 64 Go de mémoire.

Pour cette configuration, j'ai décidé de créer deux profils d'agent.


## Configuration du mini PC

| Composant | Détail |
|-----------|--------|
| **CPU** | AMD Ryzen 5 7430U (6 cœurs / 12 threads) |
| **GPU** | Aucun dédié — inférence CPU uniquement |
| **RAM** | 2 × 32 Go DDR4 |
| **Stockage** | SSD 512 Go NVMe |
| **OS** | Debian 13 amd64 |


## Configuration BIOS

- Régler **« State on power loss »** sur **« Power on »**
  (redémarrage automatique après coupure de courant)


## Installation de l'OS

Tout comme mon premier setup, j'ai installé une Debian 13.

**Partitionnement :**

| Partition | Taille |
|-----------|--------|
| EFI | 512 Mo |
| Swap | 16 Go |
| Racine (ext4) | Reste de l'espace |

Concernant **tasksel**, je n'ai installé que le strict nécessaire :

- Outils de base du système
- Serveur SSH

Je n'ai pas défini de mot de passe root, je préfère utiliser `sudo` via un
compte personnel. **Attention :** autoriser `sudo` sur le compte dédié à
l'agent Hermes était une erreur. Le timestamp caching de `sudo` me l'a
rappelé.


## Optimisation

J'ai optimisé la partition racine dans `/etc/fstab` :

```
UUID=<UUID> / ext4 errors=remount-ro,noatime 0 1
```

- **`noatime`** — ne pas mettre à jour l'heure d'accès à chaque lecture

Pour être sûr de la bonne prise en charge du matériel, j'ai installé les
paquets suivants :

```bash
sudo apt install firmware-amd-graphics firmware-linux firmware-linux-nonfree
```


## Installation des prérequis

```bash
sudo apt install curl git openjdk-25-jre-headless
```

- **curl** — téléchargements
- **git** — versionning
- **openjdk-25-jre-headless** — requis pour Signal


## Installation de l'agent

Tout est documenté dans la
[doc officielle](https://hermes-agent.nousresearch.com/docs/getting-started/installation).

J'ai choisi d'installer l'agent depuis son utilisateur dédié **hermes**.
**L'installation via `sudo` est à éviter !**

```bash
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
```

Tout s'est déroulé sans accroc. À la fin du script d'installation, le script
de configuration se lance automatiquement. J'ai choisi une configuration rapide
pour commencer : choisir un modèle et une messagerie pour la gateway.


## Modèle local

*Note : cette section a été réécrite avec le recul. La réalité a été plus
complexe que ce que j'imaginais au départ.*

### Première tentative : Ollama

J'ai installé [Ollama](https://ollama.com) :

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

Puis téléchargé un premier modèle :

```bash
ollama pull qwen2.5:14b
```

### Les obstacles rencontrés

**1. Contexte insuffisant.** Hermes Agent exige **64 000 tokens de contexte
minimum**. Ollama bride le contexte selon la RAM détectée — avec 16 Go, je
n'avais que 4 096 tokens. Même après l'upgrade à 64 Go, il a fallu forcer
la variable `OLLAMA_CONTEXT_LENGTH=65536` dans le service systemd.

**2. Les modèles Qwen2.5 sont bridés.** Les GGUF packagés par Ollama pour
les modèles Qwen2.5 (7b, 14b, 32b) ont leur contexte câblé en dur à
**32 768 tokens** — impossible à contourner, même avec un Modelfile.
`ollama show qwen2.5:14b --verbose` confirme la limite.

**3. Le bug stream + outils.** Le endpoint `/v1/chat/completions` d'Ollama
plante quand on combine `stream=true` avec des définitions d'outils.
Hermes envoie ~25 outils natifs et force `stream=True` → le bug est
déclenché à chaque requête. L'API entière devient inerte, seul un
redémarrage du service la débloque. [issue GitHub](https://github.com/NousResearch/hermes-agent/issues/25629)

**4. La détection de routes Hermes.** Même en configurant
`api_mode: chat_completions`, Hermes sonde des routes spécifiques à
Ollama (`/api/tags`, `/api/show`) avant d'envoyer la requête. Si elles
manquent (proxy LiteLLM), Hermes abandonne sans même tenter l'appel.

### Les modèles testés

| Modèle | RAM | Résultat |
|--------|-----|----------|
| qwen2.5:7b | 16 Go | Contexte 32K — bloqué |
| qwen2.5:14b | 16 Go | Contexte 32K — bloqué |
| mistral-small:22b | 64 Go | Contexte 128K ✅ mais lent CPU-only |
| llama3.3:70b | 64 Go | Trop lent pour un usage temps réel |

### Le test avec llama.cpp

J'ai fini par installer
[llama.cpp server](https://github.com/ggml-org/llama.cpp)
sur le GEEKOM lui-même pour contourner le bug stream+tools d'Ollama.
Compilation sans GPU (CPU-only).

J'ai téléchargé sur Hugging Face un **Carnice 9B** en Q4_K_M (~5,6 Go) — le
modèle recommandé pour Hermes à ce moment-là.

### Le verdict

**Ça répondait, mais pas dans des délais raisonnables.** Sans GPU, l'inférence
se traînait — probablement 1 à 3 tokens par seconde. Une simple requête avec
outils prenait plusieurs minutes. Trop lent pour un usage interactif avec
Hermes, mais ça prouvait que le concept tenait : un LLM local était possible.

Les modèles plus gros (mistral-small:22b, llama3.3:70b) n'avaient aucun intérêt
en CPU-only — le temps de réponse les rendait inutilisables.

### La conclusion

Le GEEKOM A5 a été recyclé en PC de travail quotidien. Il aurait pu faire
tourner des modèles pour des tâches script (analyse, résumé, traitement par
lots) mais pas pour un agent temps réel avec outils et streaming.

**Un LLM local pour un agent autonome, ça a besoin d'un GPU dédié.**
La RAM ne suffit pas.


## Création d'un profil

```bash
hermes profile create aether
aether setup
```


## Gateway

```bash
sudo loginctl enable-linger hermes
aether gateway setup
aether gateway install
```
