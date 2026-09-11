# Audit de sécurité rétroactif

Audit en lecture seule de l'historique des sessions pour vérifier si une action
passée a lu, copié, modifié ou exfiltré le contenu d'un élément sensible
(exemples : `~/.ssh/`, `~/.hermes/.env`, `~/.hermes/auth.json`).

À lancer sur une session de confiance. L'audit ne touche jamais aux fichiers
cibles : il ne fait que relire l'historique. À noter que la recherche fait
remonter les secrets dans le contexte de la session d'audit pendant le
dépouillement : c'est une ré-exposition ponctuelle, à assumer en connaissance
de cause.

## Prompt

```
Tu es chargé d'un audit de sécurité rétroactif. Objectif : déterminer si, dans
toutes les sessions passées, une action a lu, copié, modifié ou exfiltré le
contenu de l'un de ces éléments sensibles :

- ~/.ssh/ (et son contenu : id_rsa, id_ed25519, authorized_keys, known_hosts, config)
- ~/.hermes/.env
- ~/.hermes/auth.json

Collecte (à faire intégralement avant de classer quoi que ce soit)

Les secrets n'apparaissent pas que dans les messages de conversation : ils
vivent aussi dans les RESULTATS d'outils. Une recherche par défaut, limitée
aux messages, les rate.

1. Fais deux passes complémentaires sur l'historique :
   - passe FTS (session_search) : recherche par mots-clés en passant
     role_filter='tool', puis role_filter='user,assistant,tool', pour couvrir
     aussi bien les résultats d'outils que les messages.
   - passe exhaustive (sqlite3 sur ~/.hermes/state.db, table messages) : grep
     sur toutes les colonnes content, en SQL, pour ne rater aucune occurrence.
     C'est la passe qui fait foi ; la passe FTS sert à orienter, pas à conclure.

2. Pour chaque élément de la liste, dérive toi-même les variantes de recherche :
   chemin absolu, chemin relatif, chaque nom de fichier qui le compose, et les
   mots-clés que son contenu suggère (en-tête de clé, nom de variable, motif de
   valeur). Élargis tant que de nouvelles occurrences apparaissent.

3. Matérialise d'abord la liste BRUTE et exhaustive des occurrences (session,
   outil, commande, résultat), sans jugement. Ne classe et ne compte qu'une
   fois cette liste complète établie.

Classement (sur la liste brute complète)

4. Pour chaque occurrence, classe-la en deux catégories :
   - ACTION RÉELLE : l'agent a lu, écrit, copié ou exécuté quelque chose
     concernant le fichier cible.
   - MENTION PASSIVE : la chaîne apparaît seulement dans de la documentation,
     du contenu de skill, une page web récupérée, ou un exemple, sans
     opération réelle sur le fichier.

5. Cherche spécifiquement la piste "script écrit puis exécuté" :
   - Liste toutes les écritures de script (write_file, patch, heredoc, etc.).
   - Pour chaque script écrit, vérifie s'il a été exécuté (terminal,
     execute_code, python, bash) et quel en a été le résultat.
   - Vérifie si l'un de ces scripts lit ou référence l'un des trois éléments.

6. Cherche les patterns d'exfiltration réseau : base64, curl/POST vers une URL
   externe, transfer.sh, pastebin, webhook, ngrok, netcat, envoi de mail, etc.
   Signale tout cas où du contenu sensible aurait pu être envoyé à l'extérieur.

Format de sortie attendu, pour chacun des éléments sensibles :
- Nombre d'actions réelles (lecture/écriture/exécution) : N
- Détail de chaque action (session, outil, commande, résultat)
- Liste des mentions passives (à écarter, avec justification)
- Conclusion en deux niveaux distincts :
  - "aucune action réelle" ou "action réelle détectée" (exposition), en citant
    les preuves ;
  - "aucune exfiltration réseau" ou "exfiltration réseau détectée", en citant
    les preuves. Ne qualifie d'exfiltration que ce qui sort vers l'extérieur :
    une lecture ou une écriture locale sans envoi réseau est une exposition,
    pas une exfiltration.

Règles :
- Ne conclus jamais "rien trouvé" à partir d'une seule requête ; élargis les
  termes et réessaie avant de conclure.
- Ne conclus jamais sans la passe exhaustive sqlite3 : la passe FTS seule ne
  suffit pas.
- Distingue strictement une action réelle d'une simple mention dans du texte.
- Si un cas est ambigu, signale-le explicitement au lieu de le trancher
  silencieusement.
- Ne modifie, ne lis et n'exécute rien sur les fichiers cibles toi-même :
  audit en lecture seule de l'historique uniquement.
```
