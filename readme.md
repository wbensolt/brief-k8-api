# 🚀 Clients API – Kubernetes Architecture

Ce projet déploie une **API FastAPI** connectée à une base **MySQL**, le tout orchestré avec **Kubernetes** (Docker Desktop) et exposé via **Ingress NGINX**.

L’objectif est de démontrer :
- la communication entre services Kubernetes,
- la persistance des données avec des volumes,
- l’exposition HTTP via Ingress,
- l’utilisation de probes de santé.

---

## 🧱 Architecture globale

```
Utilisateur
   ↓ HTTP
Ingress NGINX
   ↓
Service api (ClusterIP)
   ↓
Pod API (FastAPI)
   ↓ SQL
Service mysql (ClusterIP)
   ↓
Pod MySQL
   ↓
PersistentVolume (données persistantes)
```

---

## 📦 Contenu du dépôt

```
K8/
├── api-deployment.yaml
├── api-service.yaml
├── mysql-deployment.yaml
├── mysql-service.yaml
├── mysql-secret.yaml
├── mysql-pvc.yaml
├── ingress.yaml
├── mysql-init-configmap.yaml
└── init.sql
```

---

## 🗄️ Base de données MySQL

### 🔹 MySQL Deployment
- Image : `sengsathit/brief-mysql:latest`
- Données stockées dans `/var/lib/mysql`
- Initialisation automatique via `init.sql`

### 🔹 Persistance
- Un **PersistentVolumeClaim (PVC)** est utilisé
- Les données survivent au redémarrage du pod

### 🔹 init.sql
Le fichier `init.sql` est monté dans :
```
/docker-entrypoint-initdb.d/
```

➡️ Il est exécuté **automatiquement au premier démarrage** du conteneur MySQL

---

## ⚙️ API FastAPI

### 🔹 Fonctionnalités
- `GET /health`
- `GET /clients`
- `GET /clients/{id}`
- `POST /clients`
- `DELETE /clients/{id}`

### 🔹 Connexion MySQL
Les variables d’environnement sont injectées **via Kubernetes**, sans modifier le code applicatif :

- `MYSQL_USER`
- `MYSQL_PASSWORD`
- `MYSQL_DB`
- `MYSQL_HOST`
- `MYSQL_PORT`

---

## 🌐 Exposition avec Ingress

### Ingress NGINX

L’API est exposée via le chemin :
```
/brief
```

Exemple :
```
http://localhost/brief/clients
```

L’Ingress utilise une règle de **rewrite** pour transmettre les requêtes à l’API sans le préfixe `/brief`.

---

## ❤️ Health & Readiness Probe

Le pod API expose :
```
GET /health
```

Kubernetes utilise cette route pour :
- vérifier que l’API est prête
- inclure ou exclure le pod du Service

Un pod non prêt **ne reçoit aucun trafic**.

---

## 🔐 Secrets Kubernetes

Les informations sensibles (mot de passe MySQL, utilisateur, base) sont stockées dans :

```
mysql-secret.yaml
```

➡️ Elles ne sont **jamais hardcodées dans le code**.

---

## 🧪 Tests des endpoints

### Exemple POST (Postman / curl)

```json
{
  "first_name": "John",
  "last_name": "Doe",
  "email": "john.doe@example.com"
}
```

### Suppression
```
DELETE /brief/clients/1
```

---

## 🧠 Points clés Kubernetes démontrés

- Communication inter-services via DNS (`mysql`)
- Persistance des données avec PVC
- Séparation application / configuration
- Health checks et readiness probes
- Exposition HTTP avec Ingress

---

## ✅ Prérequis

- Docker Desktop avec Kubernetes activé
- kubectl configuré
- Ingress NGINX installé

---

## 🚀 Déploiement

```bash
kubectl apply -f K8/
```

Vérification :
```bash
kubectl get pods -n brief
kubectl get svc -n brief
kubectl get ingress -n brief
```

---

## 👨‍💻 Auteur

**Wael Bensoltana**  
Développeur Data / IA – Java – Python – Kubernetes

---

🎯 Ce projet sert de **POC pédagogique** pour comprendre Kubernetes en pratique.

