# Architecture technique — Dan.09

But

Fournir une vue claire, modulaire et évolutive de l'architecture de Dan.09 pour guider les développements, les intégrations de modèles et les préoccupations de sécurité.

Vue d'ensemble

Dan.09 est conçu en couches indépendantes :

- Interface utilisateur (Web / mobile / desktop)
- Backend d'orchestration
- Couche fournisseurs (adapters pour chaque modèle/service)
- Stockage et mémoire (documents, index sémantique, historique)
- Moteur d'automatisations et planificateur
- Système de sécurité et permissions (couche indépendante)
- Observabilité (logs, audit, métriques)

Composants détaillés

1. Frontend
   - Applications : web (React/Vite), desktop (Electron/Tauri) ou mobile (PWA/ReactNative).
   - Responsiveness, accessibilité, support multilingue (FR/EN/HT).

2. Backend
   - API REST/GraphQL (ex: FastAPI / Node+Express / NestJS).
   - Gestion des sessions, des utilisateurs et des permissions.
   - Orchestrateur des appels aux fournisseurs de modèles.

3. Couche fournisseurs (Model Adapter Layer)
   - Adaptateurs par fournisseur (OpenAI, Anthropic, Mistral, Google, Ollama, etc.).
   - Registre dynamique des modèles disponibles et de leurs capacités.
   - Sélection et routage automatique/manuel selon la tâche.
   - Gestion des quotas, coûts et fallback en cas d'échec.

4. Mémoire et documents
   - Stockage des préférences, historique et mémoire par projet.
   - Index sémantique (RAG) : vector store (ex: Pinecone, Milvus, Weaviate) ou run local (FAISS).
   - Import/export sécurisé des documents (PDF, markdown, images).

5. Automatisations et planification
   - Moteur de tâches récurrentes et file d'attente (ex: Redis + workers).
   - Permissions granulaires pour actions autonomes.

6. Sécurité
   - Stockage chiffré des secrets (ex: Vault, chiffrement local).
   - Module «panic_mode» indépendant pour arrêter les automations.
   - Journal d'audit immuable (ou append-only) pour les actions sensibles.

7. Observabilité
   - Logs structurés, traces distribuées, métriques et alertes.

Sélection des technologies (exemples)

- Langage backend : TypeScript (Node/NestJS) ou Python (FastAPI)
- Frontend : React + Vite
- Base de données : PostgreSQL (métadonnées) + Redis (caches/queue)
- Stockage documents : S3-compatible
- Vector store : FAISS local / Weaviate / Milvus

Principes d'intégration des fournisseurs

- Chaque fournisseur a un adaptateur implémentant une interface commune (ex: sendPrompt, stream, cancel, capabilities).
- Le registre de modèles expose : nom, fournisseur, capacités (streaming, images), coûts estimés, latence observée.
- En cas d'erreur, fallback configurable : autre modèle local / distant ou retour d'erreur explicite.

Gestion des erreurs et reprise

- Réessayer automatiquement pour erreurs transitoires (backoff exponentiel).
- Ne PAS tenter d'utiliser des fonctionnalités non supportées par le modèle.
- Enregistrer les échecs et exposer un canal de diagnostic.

Mécanisme d'"Arrête tout" (panic)

- Flag global "panic_mode" stocké dans un emplacement sécurisé (DB chiffrée ou fichier chiffré local).
- Workers et orchestrateur pollent ou s'abonnent à une notification (pub/sub) pour ce flag.
- Quand panic_mode = true :
  - Toutes les nouvelles tâches sont refusées.
  - Les workers en cours effectuent un arrêt ordonné (flush logs, état) si possible, sinon arrêt brutal.
  - Un rapport d'état est généré automatiquement pour diagnostic.
- Autorisation : seul un administrateur authentifié (2FA recommandé) peut activer/désactiver.

Sécurité et séparation des privilèges

- Le noyau de sécurité est indépendant du moteur d'IA et ne peut pas être modifié via prompts.
- Les clés API utilisateurs sont toujours chiffrées et jamais stockées côté client.

Observations et prochaines étapes immédiates

- Fournir un squelette de backend minimal (ex: FastAPI) et un adaptateur d'exemple.
- Ajouter tests unitaires, CI, et documentation d'installation.

