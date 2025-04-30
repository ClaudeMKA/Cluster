# 📘 Déploiement d'une API Flask avec MySQL sous Kubernetes (Minikube)

Ce projet montre étape par étape comment déployer une API Flask connectée à MySQL dans un cluster local Kubernetes avec Minikube.

---

## 🔧 Étapes réalisées

### 1. Configuration du cluster Minikube

- Lancement de Minikube avec le profil `nouveau-projet`
```bash
minikube start -p nouveau-projet
```

- Connexion à l’environnement Docker intégré de Minikube :
```bash
eval $(minikube docker-env)
```

---

### 2. Construction de l'image Docker de l’API Flask

- Déplacement dans le dossier contenant le code :
```bash
cd chips-api-flask
```

- Contenu minimal de `requirements.txt` :
```txt
flask==2.3.3
mysql-connector-python==8.0.33
```

- Dockerfile utilisé :
```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["python", "app.py"]

EXPOSE 5000
```

- Construction de l’image dans l’environnement Docker de Minikube :
```bash
docker build -t flask-api:latest .
```

---

### 3. Déploiement dans Kubernetes

- Dossier `K8S-yaml/` contenant :
  - `namespace.yaml`
  - `mysql-deployment.yaml`
  - `api-deployment.yaml`

- Le fichier `api-deployment.yaml` inclut maintenant :
```yaml
image: flask-api:latest
imagePullPolicy: IfNotPresent
```

- Déploiement dans le namespace `chips` :
```bash
kubectl apply -f namespace.yaml
kubectl apply -n chips -f .
```

- Redémarrage du déploiement après le build :
```bash
kubectl rollout restart deployment flask-api -n chips
```

---

### 4. Résolution d’erreurs

- Erreurs rencontrées :
  - `ErrImagePull` → corrigée grâce à l’image locale et à `imagePullPolicy: IfNotPresent`
  - `ModuleNotFoundError: No module named 'flask'` → corrigée en ajoutant Flask dans `requirements.txt`
  - `ImagePullBackOff` → corrigée par suppression des pods et redéploiement

- Commandes de debug utilisées :
```bash
kubectl logs -n chips <nom-du-pod>
kubectl delete pod -n chips --all
```

---

### 5. Vérification finale

- Vérification que les pods tournent :
```bash
kubectl get pods -n chips
```

- Accès à l’API via le navigateur :
```bash
minikube service flask-api -n chips
```

---

## ✅ Résultat

- API Flask opérationnelle dans Minikube
- Connexion fonctionnelle à MySQL via les variables d’environnement
- Déploiement propre grâce aux fichiers YAML
