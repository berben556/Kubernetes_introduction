# Kubernetes learning & 3-tier Demo Project

## 1. Introduction

Ce dépôt contient :
- une branche **`formation`** : exercices Docker & Kubernetes réalisés durant la formation *OpenClassrooms – Initiez-vous à Kubernetes*.
- une branche **`master`** : mise en place d’une **architecture complète en 3-tiers** déployée sur Kubernetes.


 **Objectif du projet :**  
Apprendre et prendre en main Kubernetes
à travers le suisvi d'une formation et le déploiment d'une application web composée de :

| Tier | Technologie | Image Docker |
|------|-------------|--------------|
| Base de données | PostgreSQL | `berben556/kubernetes_demo_db` |
| API Backend | Java Spring Boot | `berben556/kubernetes_demo_api` |
| Frontend Web | Vue.js + NGINX | `berben556/kubernetes_demo_front` |

L’application fonctionne déjà en Docker, et a été portée sur **Kubernetes** avec :

- Deployments  
- Services ClusterIP  
- Secrets & ConfigMaps  
- PersistentVolume + PersistentVolumeClaim (PostgreSQL)  
- Namespace dédié  
- Tests sur **Minikube** et **K3s**

Un **Ingress** est en cours de finalisation pour router le frontend et l’API depuis l’extérieur du cluster.

---

##  2. Architecture Kubernetes
```
┌─────────────────┐
│ Frontend │ --> NGINX (port 80)
└───────▲─────────┘
│ HTTP
▼
┌─────────────────┐
│ API Spring Boot │ --> port 8080 (ClusterIP)
└───────▲─────────┘
│ JDBC
▼
┌─────────────────┐
│ PostgreSQL │ --> PV + PVC persistants
└─────────────────┘
```


**PostgreSQL stocke ses données dans un PersistentVolume** afin de conserver les données même si le pod redémarre.

---

## 3. Structure du dépôt

```
.
├── backend/ # Code + Dockerfile API
├── frontend/ # Code + Dockerfile Front web
├── database/ # Dockerfile + init scripts
└── k3s/
├── configmaps.yml
├── namespace.yml
├── ingress/
├── postgres/
│ ├── deployment.yml
│ ├── service.yml
│ ├── pv.yml
│ └── pvc.yml
├── api/
└── front/
```


Les manifests Kubernetes sont organisés par service pour être lisibles et maintenables.

---

## 4. Déploiement

### 1. Créer le namespace et appliquer les ressources
```
kubectl apply -f k3s/namespace.yml
kubectl apply -f k3s/configmaps.yml
minikubectl create secret generic postgres-password --from-literal=POSTGRES_PASSWORD=pwd -n kubernetes-demo
minikubectl create secret generic db-password --from-literal=DB_PASSWORD=pwd -n kubernetes-demo

kubectl apply -f k3s/postgres/
    pv -> pvc -> deployment -> service

kubectl apply -f k3s/api/
    deployment -> service

kubectl apply -f k3s/front/
    deployment -> service

kubectl apply -f k3s/ingress/

```

### 2. Vérifier le déploiement
```
kubectl get pods -n kubernetes-demo
kubectl get svc -n kubernetes-demo
kubectl get ingress -n kubernetes-demo

```

## 5. Ce qui fonctionne

| Fonction | État |
|----------|------|
| Pods des 3 services | ✅ OK |
| Services internes (ClusterIP) | ✅ OK |
| Base PostgreSQL persistante | ✅ OK (PV + PVC) |
| Secrets & ConfigMaps | ✅ OK |
| Accès API depuis le frontend | ✅ OK (via NodePort ou tunnel Minikube) |
| Ingress | 🔧 En cours de finalisation |

---

## 6. Travail restant / améliorations possibles

- Ingress fonctionnelle
- Redirection `/api` → backend instable avec `pathType: Prefix`
- Utilisation d'un vault pour les secrets
- Automatiser le déploiement (Helm chart)



---

## 7. Conclusion

Ce projet m'a permis :
d'acquérir une certaine compréhension des concepts clés de Kubernetes :

- namespaces
- deployments & services
- persistance avec PV/PVC
- secrets & configmaps
- ingress controllers
- debugging réseau Kubernetes
