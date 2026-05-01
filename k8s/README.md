# Kubernetes Manifests Structure

Cette structure permet une gestion claire et modulaire des ressources Kubernetes.

## Dossiers

- `namespaces/` : Définition des espaces de noms.
- `secrets/` : Données sensibles (identifiants DB, clés API). *Note: En production, utilisez un outil comme SealedSecrets ou SOPS.*
- `configmaps/` : Configurations non sensibles (scripts d'init, variables d'env statiques).
- `volumes/` : PersistentVolumeClaims et autres configurations de stockage.
- `deployments/` : Définitions des Deployments (backend, frontend, database).
- `services/` : Services pour exposer les applications en interne ou externe.

## Utilisation

Pour appliquer l'ensemble de la configuration :

```bash
kubectl apply -f k8s/namespaces/
kubectl apply -f k8s/secrets/
kubectl apply -f k8s/configmaps/
kubectl apply -f k8s/volumes/
kubectl apply -f k8s/deployments/
kubectl apply -f k8s/services/
```

Ou plus simplement (si aucun ordre strict n'est requis au-delà du namespace) :

```bash
kubectl apply -f k8s/ --recursive
```
