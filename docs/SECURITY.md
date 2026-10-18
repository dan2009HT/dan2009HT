# Sécurité — Dan.09

Principes généraux

1. Défense en profondeur : plusieurs couches de sécurité (auth, autorisations, chiffrement).
2. Principe du moindre privilège pour chaque composant et service.
3. Aucune clé API ou secret dans le code versionné ou dans des logs non chiffrés.
4. Journalisation d'audit pour toutes les opérations sensibles.

Stockage des secrets

- Utiliser un magasin de secrets chiffré (Vault, cloud KMS, ou chiffrement local avec clé dérivée d'un mot de passe maître).
- Fournir un fichier d'exemple `config.example.*` sans secrets.

Accès et permissions

- Authentification robuste (OAuth2 / JWT + refresh tokens ou session server-side).
- 2FA recommandé pour les administrateurs.
- Permissions granulaires par action (lecture mémoire, exécution d'automations, arrêt d'urgence).

Arrêt d'urgence ("Arrête tout")

Procédure technique :
1. Le backend expose un endpoint admin (protégé) ou une CLI pour activer `panic_mode`.
2. `panic_mode` = true est enregistré dans la DB chiffrée et publié sur un canal pub/sub (Redis, etc.).
3. Les workers et orchestrateurs s'abonnent et stoppent proprement leur activité.
4. Un rapport d'état est enregistré et envoyé à l'administrateur.

Procédure humaine :
- Si l'utilisateur (ou un administrateur) dit « Arrête tout », l'agent déclenche la vérification automatique et place le système en `panic_mode` jusqu'à investigation.
- Les étapes d'investigation doivent être consignées (qui, quand, pourquoi).

Remise en service

- La sortie du `panic_mode` doit être manuelle et contrôlée (vérification d'état, correctifs appliqués, tests passés).

Bonnes pratiques

- Ne pas confier la sortie d'urgence à un seul facteur d'authentification faible.
- Conserver des sauvegardes chiffrées et récentes.
- Effectuer des tests d'activation du mode d'arrêt en environnement staging.

