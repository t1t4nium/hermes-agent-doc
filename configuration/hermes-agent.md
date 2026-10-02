# Configuration par défaut

Configuration par défaut de toutes mes installations. J'ajuste ensuite sur cette base.

Appliquer au profil/bot avec `hermes profile use <profil>`.

## Mise à jour

Backup pre-update (`quick` : snapshot des fichiers critiques, `config.yaml`, `.env`, `auth.json`, cron et bases de profils, dans `state-snapshots/`) :

```bash
hermes config set updates.pre_update_backup quick
```

Suivi du canal stable pour les mises à jour :

```bash
hermes update --set-channel stable
```

## Sécurité

Masquer les secrets (clés API, tokens, mots de passe) dans les sorties d'outils (sortie du terminal, contenu de `read_file`, contenu web, résumés de sous-agents) avant leur entrée dans le contexte et les logs :

```bash
hermes config set security.redact_secrets true
```

## Approbations

Demander une approbation manuelle avant chaque commande shell dangereuse
(par exemple `rm -rf` ou `git reset --hard`) :

```bash
hermes config set approvals.mode manual
```

Définir à 300 secondes le délai de réponse à une demande d'approbation (au-delà, la commande est refusée) :

```bash
hermes config set approvals.timeout 300
```

En cron, personne ne peut répondre à une demande d'approbation : refuser
immédiatement les commandes dangereuses pour que l'agent trouve une autre
voie au lieu d'attendre la fin du timeout :

```bash
hermes config set approvals.cron_mode deny
```

Bloquer, même en mode `--yolo`, les commandes qui accèdent aux fichiers
sensibles (`.env`, `auth.json` et `.ssh`) :

```bash
hermes config set approvals.deny '["cat*.env*", "less*.env*", "more*.env*", "head*.env*", "tail*.env*", "grep*.env*", "sed*.env*", "awk*.env*", "cut*.env*", "base64*.env*", "strings*.env*", "od*.env*", "xxd*.env*", "diff*.env*", "cat*auth.json*", "less*auth.json*", "more*auth.json*", "head*auth.json*", "tail*auth.json*", "grep*auth.json*", "sed*auth.json*", "awk*auth.json*", "cut*auth.json*", "base64*auth.json*", "strings*auth.json*", "od*auth.json*", "xxd*auth.json*", "diff*auth.json*", "cat*.ssh*", "less*.ssh*", "more*.ssh*", "head*.ssh*", "tail*.ssh*", "grep*.ssh*", "sed*.ssh*", "awk*.ssh*", "cut*.ssh*", "base64*.ssh*", "strings*.ssh*", "od*.ssh*", "xxd*.ssh*", "diff*.ssh*", "cat*shaarli/client.ini*", "less*shaarli/client.ini*", "more*shaarli/client.ini*", "head*shaarli/client.ini*", "tail*shaarli/client.ini*", "grep*shaarli/client.ini*", "sed*shaarli/client.ini*", "awk*shaarli/client.ini*", "cut*shaarli/client.ini*", "base64*shaarli/client.ini*", "strings*shaarli/client.ini*", "od*shaarli/client.ini*", "xxd*shaarli/client.ini*", "diff*shaarli/client.ini*"]'
```

## Gateway

Pour permettre à l'agent d'avoir la date et l'heure des messages :

```bash
hermes config set gateway.message_timestamps.enabled true
```

## Mémoire par défaut

Je gère manuellement la mémoire par défaut (MEMORY.md et USER.md). Je préfère
donc désactiver les outils de l'agent sans désactiver la mémoire.

```bash
hermes config set agent.disabled_toolsets '["memory"]'
```

Filet de sécurité en cas de réactivation accidentelle des outils de mémoire :

```bash
hermes config set memory.write_approval true
```

## Skills

Désactiver l'archivage automatique des skills officielles :

```bash
hermes config set curator.prune_builtins false
```

Soumettre à approbation l'écriture dans les skills :

```bash
hermes config set skills.write_approval true
```

## Terminal

Définir le répertoire de travail par défaut du terminal de l'agent :

```bash
hermes config set terminal.cwd $HOME/workspace-<profil>
```

## Affichage

Afficher date et heure des messages :

```bash
hermes config set display.timestamps true
```

## Plugins

Nettoyer automatiquement les fichiers éphémères (scripts de test, sorties
temporaires, journaux de cron) créés pendant les sessions, sans action de
l'agent :

```bash
hermes plugins enable disk-cleanup
```

Ajouter un onglet Memory Wiki au tableau de bord pour parcourir les pages
par sujet et les journaux quotidiens issus de l'historique des sessions,
avec un panneau en lecture seule d'audit de la mémoire persistante
(MEMORY.md et USER.md) :

```bash
hermes plugins install hermes-memory-wiki
```

Masquer les valeurs exactes des secrets du `.env` (repérés par le nom de
variable) dans toutes les sorties d'outils et les logs, avant leur entrée
dans le contexte :

```bash
hermes plugins install t1t4nium/hermes-agent-env-secret-redactor
```

Réinstaller les skills fournies par Hermes dans leurs dossiers de catégorie
depuis la source embarquée, en une seule commande :

```bash
hermes plugins install t1t4nium/hermes-agent-reset-bundled-skills
```

Ajouter un navigateur en lecture seule pour une base de connaissances
LLM wiki dans l'application desktop (recherche plein texte, liste des pages,
rendu markdown, navigation par wikiliens et rétroliens) :

```bash
hermes plugins install t1t4nium/hermes-agent-llm-wiki-browser
```

## Providers et modèles

Définir les clés API de chaque provider dans le fichier `~/.hermes/.env` :

```bash
set +o history
echo 'NOUS_API_KEY=<CLE_API>' >> ~/.hermes/.env
echo 'AIML_API_KEY=<CLE_API>' >> ~/.hermes/.env
echo 'HF_TOKEN=<CLE_API>' >> ~/.hermes/.env
set -o history
```

Définir les providers :

```bash
hermes config set providers.nousapi.base_url https://inference-api.nousresearch.com/v1
hermes config set providers.nousapi.key_env NOUS_API_KEY
hermes config set providers.nousapi.default_model deepseek/deepseek-v4-pro
hermes config set providers.aimlapi.base_url https://api.aimlapi.com/v1
hermes config set providers.aimlapi.key_env AIML_API_KEY
hermes config set providers.aimlapi.default_model deepseek/deepseek-v4-pro
hermes config set providers.hfapi.base_url https://router.huggingface.co/v1
hermes config set providers.hfapi.key_env HF_TOKEN
hermes config set providers.hfapi.default_model deepseek-ai/DeepSeek-V4-Pro:cheapest
```

Définir le modèle par défaut :

```bash
hermes config set model.provider nousapi
hermes config set model.default deepseek/deepseek-v4-pro
```

Définir le modèle de délégation :

```bash
hermes config set delegation.provider nousapi
hermes config set delegation.model deepseek/deepseek-v4-pro
```

Définir les providers fallback :

```bash
hermes config set fallback_providers '[{"provider": "aimlapi", "model": "deepseek/deepseek-v4-pro", "base_url": "https://api.aimlapi.com/v1"}, {"provider": "hfapi", "model": "deepseek-ai/DeepSeek-V4-Pro:cheapest", "base_url": "https://router.huggingface.co/v1"}]'
```

## Tâches auxiliaires

Les tâches auxiliaires déchargent le modèle principal sur des appels ciblés
(analyse d'image, résumé de contexte, titres, etc.). Chacune se configure sur
le provider `nousapi` avec un modèle adapté à son enjeu : le modèle principal
pour ce qui touche à l'état durable, un modèle multimodal pour la vision, et
un modèle léger pour les tâches indépendantes à haut volume.

### Niveau qualité (modèle principal)

Ces tâches modifient l'état durable ou sont destructives. Elles conservent le
modèle principal `deepseek/deepseek-v4-pro`.

**compression**

Résume le milieu de la conversation quand le contexte approche la limite. Le
résumé remplace le transcript, la perte est irréversible, on garde donc le Pro
pour sa fidélité.

```bash
hermes config set auxiliary.compression.provider nousapi
hermes config set auxiliary.compression.model deepseek/deepseek-v4-pro
```

**review**

Sous-agent qui relit le diff et produit des commentaires. L'original reste
intact, mais un petit modèle rate les bugs subtils tout en ayant l'air
confiant. On garde le Pro pour la profondeur d'analyse.

```bash
hermes config set auxiliary.review.provider nousapi
hermes config set auxiliary.review.model deepseek/deepseek-v4-pro
```

**background_review**

Fork post-tour qui écrit en mémoire et corrige les skills. Cette sortie
devient de l'état durable réinjecté dans les sessions suivantes. Le Pro rejoue
la conversation depuis le cache chaud, le surcoût est faible.

```bash
hermes config set auxiliary.background_review.provider nousapi
hermes config set auxiliary.background_review.model deepseek/deepseek-v4-pro
```

**curator**

Revue périodique de l'usage des skills et consolidation éventuelle. Elle
modifie l'état durable des skills. Elle ne tourne qu'une fois par semaine,
le coût est négligeable et on privilégie la qualité.

```bash
hermes config set auxiliary.curator.provider nousapi
hermes config set auxiliary.curator.model deepseek/deepseek-v4-pro
```

**moa_aggregator**

Synthétise les réponses des modèles conseillers en une réponse finale dans le
circuit MoA. C'est le dernier maillon, celui qui fixe la qualité de la réponse.
On garde le modèle le plus fort.

```bash
hermes config set auxiliary.moa_aggregator.provider nousapi
hermes config set auxiliary.moa_aggregator.model deepseek/deepseek-v4-pro
```

### Niveau multimodal

Ces tâches exigent la vision, que `deepseek-v4-pro` (texte seul) ne sait pas
faire. Elles utilisent `minimax/minimax-m3`.

**vision**

Analyse les images et les captures d'écran. Elle impose un modèle multimodal,
ce que le cerveau principal texte seul ne peut pas assurer. `minimax-m3` est
multimodal et moins coûteux en sortie que l'alternative Gemini.

```bash
hermes config set auxiliary.vision.provider nousapi
hermes config set auxiliary.vision.model minimax/minimax-m3
```

**moa_reference**

Modèle conseiller dans le circuit MoA. La diversité compte en MoA : un
fournisseur différent de l'agrégateur décortique mieux les erreurs. `minimax-m3`
apporte cette diversité à faible coût.

```bash
hermes config set auxiliary.moa_reference.provider nousapi
hermes config set auxiliary.moa_reference.model minimax/minimax-m3
```

### Niveau léger

Tâches indépendantes, à faible enjeu ou à haut volume. Elles utilisent
`deepseek/deepseek-v4-flash-0731`.

**skills_hub**

Recherche et classement de skills. Tâche indépendante et sans état durable,
un mauvais classement est visible et corrigeable. Flash convient.

```bash
hermes config set auxiliary.skills_hub.provider nousapi
hermes config set auxiliary.skills_hub.model deepseek/deepseek-v4-flash-0731
```

**approval**

Classifieur qui décide d'approuver ou non une commande. Le code source
recommande explicitement un modèle rapide et bon marché. La décision binaire
tolère un petit modèle.

```bash
hermes config set auxiliary.approval.provider nousapi
hermes config set auxiliary.approval.model deepseek/deepseek-v4-flash-0731
```

**mcp**

Raisonnement pour router les appels d'outils MCP. Aiguillage à faible enjeu,
consommé immédiatement. Flash fait l'affaire.

```bash
hermes config set auxiliary.mcp.provider nousapi
hermes config set auxiliary.mcp.model deepseek/deepseek-v4-flash-0731
```

**title_generation**

Génère le titre de la session après le premier échange. Tâche cosmétique et
fréquente. Flash réduit le coût sans impact perceptible.

```bash
hermes config set auxiliary.title_generation.provider nousapi
hermes config set auxiliary.title_generation.model deepseek/deepseek-v4-flash-0731
```

**memory_query_rewrite**

Réécrit une requête en langage naturel en requête de recherche mémoire. Appel
court et borné. Flash suffit.

```bash
hermes config set auxiliary.memory_query_rewrite.provider nousapi
hermes config set auxiliary.memory_query_rewrite.model deepseek/deepseek-v4-flash-0731
```

**tts_audio_tags**

Insertion d'étiquettes audio cachées pour le TTS Gemini. Inactif tant que le
TTS n'est pas Gemini. Flash par défaut, sans enjeu.

```bash
hermes config set auxiliary.tts_audio_tags.provider nousapi
hermes config set auxiliary.tts_audio_tags.model deepseek/deepseek-v4-flash-0731
```

**triage_specifier**

Étoffe une carte kanban en triage en spec complète. Le code source indique
qu'un modèle bon marché suffit. Flash est adapté.

```bash
hermes config set auxiliary.triage_specifier.provider nousapi
hermes config set auxiliary.triage_specifier.model deepseek/deepseek-v4-flash-0731
```

**kanban_decomposer**

Décompose une tâche en graphe de sous-tâches JSON. Travail mécanique qui produit
surtout plus de tokens. Flash suffit largement.

```bash
hermes config set auxiliary.kanban_decomposer.provider nousapi
hermes config set auxiliary.kanban_decomposer.model deepseek/deepseek-v4-flash-0731
```

**profile_describer**

Décrit un profil en une ou deux phrases. Tâche courte sans état durable. Flash
convient.

```bash
hermes config set auxiliary.profile_describer.provider nousapi
hermes config set auxiliary.profile_describer.model deepseek/deepseek-v4-flash-0731
```

**goal_judge**

Évalue la satisfaction d'un objectif `/goal` et rédige le contrat associé.
Appels JSON à enjeu modéré et non destructifs. Flash fait l'affaire.

```bash
hermes config set auxiliary.goal_judge.provider nousapi
hermes config set auxiliary.goal_judge.model deepseek/deepseek-v4-flash-0731
```

**monitor**

Attribue un score de 0 à 10 aux mails importants. Haut volume, le code source
recommande un petit modèle. Flash est adapté.

```bash
hermes config set auxiliary.monitor.provider nousapi
hermes config set auxiliary.monitor.model deepseek/deepseek-v4-flash-0731
```

Les clés `auxiliary.web_extract` et `auxiliary.session_search` n'existent plus :
les outils correspondants n'utilisent plus de modèle auxiliaire. Ne pas les
définir.