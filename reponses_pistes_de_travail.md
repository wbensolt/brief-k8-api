# Réponses – Pistes de travail Kubernetes

Ce document regroupe des réponses claires et directement liées à l’architecture mise en place dans ce projet : **FastAPI + MySQL + Kubernetes + Ingress**.

---

## 1. Volume et persistance

### Quel est le rôle d’un volume dans un déploiement Kubernetes ?

Un **volume** permet de stocker des données **en dehors du cycle de vie d’un conteneur**.
Dans notre architecture, le volume est utilisé pour :
- conserver les **données MySQL** (tables, enregistrements clients)
- éviter toute perte de données lors d’un redémarrage du pod MySQL

Sans volume, toutes les données seraient perdues à chaque recréation du pod.

---

### Que signifie la mention `storageClassName` dans un PVC, et que peut-elle impliquer côté cloud ?

`storageClassName` indique **quel type de stockage** Kubernetes doit utiliser.

Cela peut impliquer :
- le type de disque (SSD, HDD)
- les performances (IOPS, latence)
- le fournisseur cloud (Azure Disk, AWS EBS, GCP Persistent Disk)
- la création automatique du stockage

👉 Dans le cloud, cette valeur déclenche souvent la **création dynamique d’un disque** facturé.

---

### Que se passe-t-il si le pod MySQL disparaît ?

Si le pod MySQL est supprimé ou redémarré :
- Kubernetes recrée automatiquement un nouveau pod
- le **PersistentVolumeClaim est réutilisé**
- les données stockées dans le volume sont **restaurées automatiquement**

👉 Résultat : **les données MySQL sont conservées**.

---

### Qu’est-ce qui relie un PersistentVolumeClaim à un volume physique ?

Le lien est assuré par :
- le **PersistentVolume (PV)**
- le **PVC**
- la **StorageClass** (si provisionnement dynamique)

Flux logique :

PVC → StorageClass → PersistentVolume → Stockage physique (disque)

---

### Comment le cluster gère-t-il la création ou la suppression du stockage sous-jacent ?

Cela dépend du mode :

- **Provisionnement dynamique** :
  - le cluster crée automatiquement le disque
  - le disque est supprimé ou conservé selon la politique (`Delete` ou `Retain`)

- **Provisionnement statique** :
  - le disque existe déjà
  - Kubernetes se contente de l’attacher

👉 Dans un environnement cloud, ces opérations sont souvent **automatisées et facturées**.

---

## 2. Ingress et health probe

### À quoi sert un Ingress dans Kubernetes ?

Un **Ingress** permet d’exposer une application Kubernetes **via HTTP/HTTPS** depuis l’extérieur du cluster.

Dans ce projet, l’Ingress permet :
- d’accéder à l’API FastAPI depuis `http://localhost/brief/...`
- de router les requêtes vers le service `api`

---

### Quelle différence y a-t-il entre un Ingress et un Ingress Controller ?

- **Ingress** :
  - une **ressource Kubernetes** (règles de routage)

- **Ingress Controller** :
  - le composant qui **applique réellement ces règles**
  - ex : NGINX Ingress Controller

👉 Sans Ingress Controller, un Ingress ne fonctionne pas.

---

### À quoi sert un health probe dans une architecture de déploiement ?

Un **health probe** permet à Kubernetes de vérifier si une application est :
- démarrée correctement
- capable de répondre aux requêtes

Dans notre API :
- le probe appelle `/health`
- si la réponse est `200 OK`, le pod est considéré comme sain

---

### Quelle est la relation entre le chemin du probe et les routes exposées par l’application ?

Le chemin du probe doit :
- exister dans l’application
- répondre rapidement
- ne pas dépendre de services externes (ex : base de données)

Dans FastAPI :
```python
@app.get("/health")
def health_check():
    return {"status": "ok"}
```

👉 Si ce chemin n’existe pas, le pod sera marqué **non prêt**.

---

### Comment mettre en place un chemin de préfixe (ex. `/brief`) dans l’Ingress ?

Dans l’Ingress :
- on définit un chemin `/brief(/|$)(.*)`
- on réécrit l’URL avec `rewrite-target`

Exemple :
```
/brief/clients → /clients
```

---

### Quelle configuration doit être ajustée dans l’application ?

L’application FastAPI doit définir :

```python
ROOT_PATH = os.getenv("ROOT_PATH", "")
app = FastAPI(root_path=ROOT_PATH)
```

Et dans Kubernetes :
- définir la variable d’environnement `ROOT_PATH=/brief`

👉 Cela garantit que l’application comprend qu’elle est servie derrière un préfixe.

---

### Comment le contrôleur d’ingress décide-t-il si un service est “sain” ?

Le contrôleur d’ingress s’appuie sur :
- les **readiness probes** des pods
- l’état du **service Kubernetes**

Si :
- aucun pod n’est prêt
- ou si les probes échouent

👉 le trafic **n’est plus routé** vers ce service.

---

## Conclusion

Cette architecture illustre :
- la **séparation des responsabilités** (API / DB)
- la **persistance des données** via PVC
- l’exposition contrôlée via **Ingress**
- la supervision automatique grâce aux **health probes**

Elle correspond à une architecture Kubernetes **réaliste et professionnelle**.

