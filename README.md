# k8s-gitops
Infrastructure Kubernetes avec GitOps

# Déploiement de Nginx et PostgreSQL sur Kubernetes

Ce README explique comment déployer Nginx et PostgreSQL dans un cluster Kubernetes étape par étape.

## Prérequis

- Un cluster Kubernetes fonctionnel (Docker Desktop ou autre)
- `kubectl` installé et configuré
- Les fichiers manifest YAML présents dans les dossiers `components/nginx/` et `components/postgres/`

## Structure des fichiers

```
components/
├── nginx/
│   ├── nginx-namespace.yml
│   ├── nginx-pod.yml
│   └── nginx-service.yml
└── postgres/
    ├── postgres-namespace.yml
    ├── postgres-volume.yml
    ├── postgres-pvc.yml
    ├── postgres-configmap.yml
    ├── postgres-deployment.yml
    └── postgres-service.yml
```

## Déploiement de Nginx

### 1. Création du namespace Nginx

```bash
kubectl apply -f components/nginx/nginx-namespace.yml
```

### 2. Déploiement du pod Nginx

```bash
kubectl apply -f components/nginx/nginx-pod.yml
```

### 3. Création du service Nginx

```bash
kubectl apply -f components/nginx/nginx-service.yml
```

### 4. Vérification du déploiement Nginx

```bash
# Vérifier que le pod est en cours d'exécution
kubectl get pods -n nginx

# Vérifier que le service est créé
kubectl get services -n nginx
```

### 5. Accès à Nginx

```bash
kubectl port-forward service/nginx-service 8080:21000 -n nginx
```

Ouvrez votre navigateur et accédez à [http://localhost:8080](http://localhost:8080).

## Déploiement de PostgreSQL

### 1. Création du namespace PostgreSQL

```bash
kubectl apply -f components/postgres/postgres-namespace.yml
```

### 2. Création du volume persistant et de sa réclamation

```bash
kubectl apply -f components/postgres/postgres-volume.yml
kubectl apply -f components/postgres/postgres-pvc.yml
```

### 3. Création du ConfigMap pour les informations d'identification

```bash
kubectl apply -f components/postgres/postgres-configmap.yml
```

> **Note de sécurité**: En production, utilisez des Kubernetes Secrets au lieu de ConfigMaps pour stocker les mots de passe.

### 4. Déploiement de PostgreSQL

```bash
kubectl apply -f components/postgres/postgres-deployment.yml
```

### 5. Création du service PostgreSQL

```bash
kubectl apply -f components/postgres/postgres-service.yml
```

### 6. Vérification du déploiement PostgreSQL

```bash
# Vérifier que le pod est en cours d'exécution
kubectl get pods -n postgres

# Vérifier que le service est créé
kubectl get services -n postgres

# Vérifier tous les composants du déploiement
kubectl get all -n postgres
```

### 7. Accès à PostgreSQL

```bash
kubectl port-forward service/postgres-service 5432:5432 -n postgres
```

## Connexion à PostgreSQL

### Via ligne de commande

```bash
PGPASSWORD=ps_password psql -h localhost -p 5432 -U ps_user -d ps_db
```

### Via pgAdmin

1. Ouvrez pgAdmin et créez une nouvelle connexion avec les paramètres suivants:
   - Nom: Kubernetes PostgreSQL
   - Hôte: localhost
   - Port: 5432
   - Base de données: ps_db
   - Utilisateur: ps_user
   - Mot de passe: ps_password

## Test des déploiements

### Test de Nginx

Ouvrez votre navigateur à l'adresse [http://localhost:8080](http://localhost:8080) pendant que le port-forward est actif.

### Test de PostgreSQL

Une fois connecté à PostgreSQL, exécutez quelques requêtes SQL:

```sql
-- Vérifier la version
SELECT version();

-- Créer une table de test
CREATE TABLE test (id SERIAL PRIMARY KEY, name VARCHAR(50));

-- Insérer des données
INSERT INTO test (name) VALUES ('test data');

-- Interroger les données
SELECT * FROM test;
```

## Dépannage

### Problèmes courants avec Nginx

#### Vérification des logs Nginx

```bash
kubectl logs -n nginx nginx-pod
```

#### Vérification de la configuration du pod Nginx

```bash
kubectl describe pod nginx-pod -n nginx
```

### Problèmes courants avec PostgreSQL

#### Port déjà utilisé

Si vous rencontrez une erreur indiquant qu'un port est déjà utilisé:

```
Unable to listen on port XXXX: Listeners failed to create with the following errors: [unable to create listener: Error listen tcp4 127.0.0.1:XXXX: bind: address already in use]
```

Solution: Utilisez un port local différent pour le port-forward:

```bash
kubectl port-forward service/SERVICE_NAME LOCAL_PORT:TARGET_PORT -n NAMESPACE
```

#### Erreur d'authentification PostgreSQL

Si vous rencontrez une erreur d'authentification lors de la connexion à PostgreSQL, vérifiez:

1. Que les informations d'identification dans le ConfigMap correspondent à celles utilisées pour la connexion
2. Que le ConfigMap est correctement référencé dans le déploiement
