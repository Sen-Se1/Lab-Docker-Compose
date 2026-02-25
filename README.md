# TaskMaster Pro - Docker Compose Lab

Une application web trois-tiers (Frontend, Backend, Database) déployée avec Docker Compose, conçue pour démontrer les principes du développement moderne et du déploiement conteneurisé.

## 🚀 Architecture

L'application suit une structure classique à trois niveaux :
1.  **Frontend** : Serveur Nginx servant des fichiers HTML/CSS/JS statiques avec un design moderne (Glassmorphism).
2.  **Backend** : API REST Node.js/Express connectée à PostgreSQL.
3.  **Database** : Instance PostgreSQL pour la persistance des données.

## 🛠️ Fonctionnalités implémentées

- **CRUD complet** : Ajout, consultation et suppression de tâches.
- **Multi-stage Build** : Dockerfile backend optimisé pour la taille et la sécurité.
- **Isolation réseau** : Réseaux séparés (`backend-net`, `frontend-net`) pour isoler la base de données.
- **Gestion d'environnement** : Configuration centralisée via un fichier `.env`.
- **Healthchecks** : Monitoring de l'état de santé de chaque conteneur.
- **Sécurité** : Processus s'exécutant avec des utilisateurs non-root.
- **Persistance** : Volume nommé pour les données PostgreSQL.

## 📋 Prérequis

- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/install/)

## 🚀 Installation et Lancement

1.  **Cloner le projet** (ou copier les fichiers).
2.  **Configurer l'environnement** :
    Vérifiez le fichier `.env` à la racine :
    ```env
    DB_NAME=taskdb
    DB_USER=taskuser
    DB_PASSWORD=taskpass
    # ... autres variables
    ```
3.  **Lancer l'application** :
    ```bash
    docker-compose up --build
    ```
4.  **Accéder à l'interface** :
    Ouvrez votre navigateur sur [http://localhost](http://localhost).

## 📂 Structure du projet

```text
.
├── backend/            # Code source de l'API Node.js
│   ├── Dockerfile      # Build multi-stage optimisé
│   └── index.js        # Logique de l'API
├── frontend/           # Code source du site web
│   ├── Dockerfile      # Serveur Nginx (non-root)
│   ├── nginx.conf      # Configuration du reverse proxy
│   └── index.html      # UI Premium
├── db/                 # Scripts de base de données
│   └── init.sql        # Initialisation du schéma
├── .env                # Variables d'environnement
├── .gitignore          # Fichiers ignorés par Git
└── docker-compose.yml  # Orchestration des services
```

## 🔐 Bonnes pratiques DevOps appliquées

- **Images légères** : Utilisation de `node:18-alpine` et `nginx:alpine`.
- **User non-root** : Les conteneurs ne s'exécutent pas avec les droits root.
- **Resource Limits** : CPU limité à 0.5 et RAM à 512Mo par service.
- **Volumes** : Persistance des données d'inventaire garantissant qu'aucune donnée n'est perdue au redémarrage.
