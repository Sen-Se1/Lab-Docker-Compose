# ☸️ Kubernetes Orchestration - Deep Dive

Ce dossier contient les manifests organisés pour déployer l'application sur un cluster Kubernetes. La structure a été conçue pour être modulaire, sécurisée et résiliente.

## 📂 Structure des Manifests

La configuration est divisée par type de ressource pour une gestion simplifiée :

- `/namespaces` : Isolation de l'application dans `task-app`.
- `/secrets` : Centralisation des données sensibles (`app-secrets`).
- `/configmaps` : Scripts d'initialisation de la base de données.
- `/volumes` : Définition du stockage persistant (PVC).
- `/deployments` : Déploiements du Frontend et du Backend, et **StatefulSet** pour Postgres.
- `/services` : Exposition réseau des composants (ClusterIP et NodePort).

---

## 🛠️ Concepts Avancés Implémentés

### 🏗️ StatefulSet pour la Database
Contrairement à un Deployment classique, le **StatefulSet** est utilisé pour Postgres car il garantit :
- Une identité réseau stable.
- Une gestion ordonnée du déploiement et des mises à jour.
- Un lien permanent avec son volume persistant.

### 🔄 Stratégie RollingUpdate (Zero Downtime)
Les déploiements sont configurés avec :
- `maxSurge: 1` : Un nouveau Pod est créé avant de supprimer l'ancien.
- `maxUnavailable: 0` : Garantit qu'aucune interruption n'est autorisée pendant la mise à jour.
Cela permet de mettre à jour les images (ex: passer de `1.0` à `1.1`) sans aucune coupure pour l'utilisateur.

### 🛡️ SecurityContext (Non-Root)
Chaque Pod est durci pour limiter les risques de sécurité :
- `runAsNonRoot: true` : Interdiction d'exécuter en tant que root.
- `runAsUser`: UID spécifique par service (1000 pour Backend, 101 pour Frontend, 999 pour Postgres).
- `fsGroup`: Défini pour assurer que les volumes montés sont accessibles par l'utilisateur non-root.

### 📈 Gestion des Ressources & Santé
- **Limits & Requests** : Définies pour éviter qu'un conteneur ne déstabilise le cluster.
- **Readiness Probes** : Kubernetes attend que l'app soit prête avant d'envoyer du trafic.
- **Liveness Probes** : Redémarrage automatique du conteneur s'il ne répond plus.

---

## 🚀 Déploiement

Pour déployer l'intégralité du stack de manière récursive :

```powershell
# Créer l'espace de noms d'abord
kubectl apply -f k8s/namespaces/

# Appliquer tout le reste récursivement
kubectl apply -f k8s -R
```

### Vérifier l'état :
```powershell
kubectl get all -n task-app
```
