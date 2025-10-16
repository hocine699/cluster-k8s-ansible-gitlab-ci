# Pipeline Déploiement d'Applications Kubernetes

Ce projet automatise le déploiement d'applications sur un cluster Kubernetes existant via GitLab CI.

## 🎯 Objectif

**Déploiement applicatif uniquement** - Ce repo ne déploie PAS d'infrastructure Kubernetes. Il déploie des applications sur un cluster K8s déjà existant.

## 📁 Structure du projet

```
k8s-app-deployment/
├── .gitlab-ci.yml           # Pipeline GitLab CI applicatif minimal
├── test-deployment.yaml     # Application nginx de test
├── README.md               # Cette documentation
└── .gitignore              # Fichiers ignorés Git
```

## 🚀 Fonctionnalités

- **Pipeline léger** : 87 lignes (vs 600+ pour infrastructure)
- **Déploiement automatique** : sur branches `test-*`
- **Déploiement manuel** : sur branche `main`
- **Test connectivité** : ping + port 6443
- **Support SSH** : récupération kubeconfig automatique
- **Fallback** : instructions manuelles si SSH échoue

## 📋 Prérequis

### Cluster Kubernetes existant
- Master : `192.168.1.72`
- Workers : `192.168.1.73`, `192.168.1.74`
- API Server accessible sur port `6443`

### GitLab CI Variables
Configurez dans GitLab → Settings → CI/CD → Variables :

| Variable | Type | Valeur | Description |
|----------|------|--------|-------------|
| `SSH_PRIVATE_KEY` | Variable | Clé SSH privée | Accès au cluster via SSH |

## 🔧 Utilisation

### Déploiement automatique (branches test-*)
```bash
git checkout -b test-nginx-deployment
git push origin test-nginx-deployment
# → Pipeline s'exécute automatiquement
```

### Déploiement manuel (main)
1. Merger dans `main`
2. Aller dans GitLab → Pipelines
3. Cliquer "Play" sur `deploy_app_manual`

### Déploiement manuel direct
```bash
# Sur le master K8s
kubectl apply -f test-deployment.yaml
kubectl get pods -l app=nginx-test
kubectl get svc nginx-test-service

# Accès application
curl http://192.168.1.72:30090
```

## 📦 Applications disponibles

### **Nginx (test-deployment.yaml)**
- **Type** : Serveur web de test
- **Replicas** : 3 pods
- **Port** : 30090
- **Ressources** : 64Mi-128Mi RAM, 250m-500m CPU
- **Accès** : http://192.168.1.72:30090
- **Déclencheur** : branches `test-*`

### **Redis (redis-deployment.yaml)**
- **Type** : Cache/Base de données en mémoire
- **Replicas** : 1 pod
- **Port** : 30379
- **Ressources** : 64Mi-128Mi RAM, 250m-500m CPU
- **Accès** : redis-cli -h 192.168.1.72 -p 30379
- **Déclencheur** : branches `redis-*`

## 🛠️ Architecture Pipeline

```
GitLab CI Runner
    ↓
Test connectivité cluster
    ↓  
SSH → Récupération kubeconfig
    ↓
kubectl apply manifest
    ↓
Application déployée
```

## 📊 Monitoring

Une fois déployé, surveillez :
- **Pods** : `kubectl get pods -l app=nginx-test`
- **Service** : `kubectl get svc nginx-test-service`
- **Logs** : `kubectl logs -l app=nginx-test`

## 🔍 Troubleshooting

### SSH échoue
1. Vérifiez variable `SSH_PRIVATE_KEY` dans GitLab
2. Testez SSH manuel : `ssh hocine@192.168.1.72`
3. Utilisez déploiement manuel avec instructions

### Port 30090 occupé
```bash
kubectl get svc --all-namespaces | grep 30090
kubectl delete svc <service-conflit>
```

### Pods en erreur
```bash
kubectl describe pod <pod-name>
kubectl logs <pod-name>
```

## 🎯 Évolution

Pour ajouter d'autres applications :
1. Créez nouveaux manifests YAML
2. Ajoutez jobs dans `.gitlab-ci.yml`
3. Adaptez les variables selon vos besoins

---

**Note** : Ce repo était auparavant utilisé pour déployer l'infrastructure K8s complète avec Ansible. Il a été simplifié pour se concentrer uniquement sur le déploiement d'applications.
├── k8s-monitoring-stack.yaml        # Stack monitoring (Node Exporter, kube-state-metrics)
├── test-deployment.yaml             # Application de test
├── cleanup-playbook.yml             # Nettoyage automatisé
├── integrate-monitoring.sh          # Intégration Prometheus/Grafana
├── prometheus-alerts.yml            # Règles d'alerte
├── deploy-runner-synology.sh        # Déploiement GitLab Runner
├── docker-compose.gitlab-runner.yml # Runner Docker Compose
├── GITLAB_CI_SETUP.md              # Guide GitLab CI
├── PROMETHEUS_INTEGRATION.md        # Guide monitoring
├── DASHBOARD_GUIDE.md               # Guide dashboard
└── README.md                        # Cette documentation
```

## 🚀 Fonctionnalités

✅ **Déploiement automatisé** via GitLab CI sur Synology NAS  
✅ **Cluster Kubernetes 1.28.2** (1 master + 2 workers)  
✅ **Containerd** comme runtime de conteneurs  
✅ **Flannel CNI** pour le réseau inter-pods  
✅ **Dashboard Kubernetes** avec interface web  
✅ **Monitoring intégré** (Node Exporter, kube-state-metrics)  
✅ **Intégration Prometheus/Grafana** existant  
✅ **Pipeline GitLab CI** complète avec 5 stages  
✅ **Scripts d'intégration** automatisés  

## ⚙️ Déploiement rapide

### **1. Configuration GitLab Variables**
```bash
# Dans GitLab: Settings > CI/CD > Variables
K8S_MASTER_IP = 192.168.1.10
K8S_WORKER1_IP = 192.168.1.11  
K8S_WORKER2_IP = 192.168.1.12
SSH_PRIVATE_KEY = [contenu de votre clé privée SSH]
```

### **2. Préparation SSH**
```bash
# Copier votre clé publique sur vos VMs
ssh-copy-id hocine@192.168.1.72
ssh-copy-id hocine@192.168.1.11
ssh-copy-id hocine@192.168.1.12
```

### **3. Déploiement automatique**
```bash
git push origin main
# → Pipeline GitLab CI se déclenche automatiquement
```

## 🌐 Accès aux services

### **Interfaces déployées automatiquement**

| Service | URL | Accès |
|---------|-----|-------|
| **Dashboard K8s** | `https://[IP_MASTER]:30443` | Token depuis artefacts GitLab |
| **App test nginx** | `http://[IP_MASTER]:30080` | Direct |
| **Node Exporter** | `http://[IP_MASTER]:9100/metrics` | Prometheus |
| **kube-state-metrics** | `http://[IP_MASTER]:30080/metrics` | Prometheus |

### **Récupération des accès**
```bash
# Télécharger les artefacts GitLab CI
# Contient: kubeconfig/, dashboard-info/, monitoring-config/

# kubeconfig pour kubectl
export KUBECONFIG=./kubeconfig/config
kubectl get nodes

# Token dashboard K8s
cat dashboard-info/dashboard-token.txt

# Configuration Prometheus
cat monitoring-config/prometheus-k8s-jobs.yml
```

## 📊 Monitoring intégré

### **Stack déployée sur K8s**
- **Node Exporter** (DaemonSet) - Métriques nœuds
- **kube-state-metrics** (Deployment) - Métriques K8s
- **ServiceAccount + Token** - Auth Prometheus

### **Intégration Prometheus/Grafana NAS**
```bash
# Script d'intégration automatique
./integrate-monitoring.sh [master_ip] [nas_ip]

# Ou récupérer config depuis GitLab CI (job deploy_monitoring)
# monitoring-config/prometheus-k8s-jobs.yml
```

### **Dashboards Grafana recommandés**
- Kubernetes Cluster Monitoring (ID: 7249)
- Kubernetes Pod Monitoring (ID: 6417)  
- Node Exporter Full (ID: 1860)
- Kubernetes Deployment (ID: 8588)

## 🔧 Pipeline GitLab CI

### **5 Stages automatisés**
1. **validate** - Validation syntaxe + connectivité SSH
2. **prepare** - Préparation environnement
3. **deploy-cluster** - Déploiement K8s (common→containerd→kubernetes→master→workers)
4. **verify** - Vérification cluster + dashboard
5. **deploy-app** - Applications (dashboard auto, monitoring manuel, test app manuel)

### **Jobs manuels disponibles**
- `deploy_monitoring` - Stack monitoring pour Prometheus/Grafana
- `deploy_test_app` - Application nginx de test  
- `cleanup_cluster` - Nettoyage complet du cluster

## 📋 Guides détaillés

- **`GITLAB_CI_SETUP.md`** - Configuration GitLab CI sur Synology
- **`PROMETHEUS_INTEGRATION.md`** - Intégration monitoring existant
- **`DASHBOARD_GUIDE.md`** - Utilisation dashboard Kubernetes

## 🛠️ Déploiement manuel (optionnel)

Si vous préférez déployer sans GitLab CI :

```bash
# Modifier l'inventaire avec vos IPs
nano inventory.yml

# Test de connectivité
ansible all -i inventory.yml -m ping

# Déploiement complet
ansible-playbook -i inventory.yml site.yml

# Ou par étapes
ansible-playbook -i inventory.yml site.yml --tags common
ansible-playbook -i inventory.yml site.yml --tags containerd
ansible-playbook -i inventory.yml site.yml --tags kubernetes
ansible-playbook -i inventory.yml site.yml --limit masters
ansible-playbook -i inventory.yml site.yml --limit workers
```

## 🚨 Dépannage

### **Problèmes courants**

1. **Nœuds en état NotReady**
   ```bash
   kubectl describe node <node-name>
   kubectl get pods -n kube-system  # Vérifier flannel
   ```

2. **Pipeline GitLab CI échoue**
   ```bash
   # Vérifier les variables CI/CD
   # Tester SSH manuellement
   ssh hocine@192.168.1.72 "uptime"
   ```

3. **Dashboard inaccessible**
   ```bash
   kubectl get pods -n kubernetes-dashboard
   kubectl get service kubernetes-dashboard -n kubernetes-dashboard
   ```

4. **Monitoring non fonctionnel**
   ```bash
   kubectl get pods -n kube-system -l app=node-exporter
   curl http://192.168.1.10:9100/metrics  # Test Node Exporter
   ```

### **Nettoyage complet**
```bash
# Via GitLab CI (job manuel)
cleanup_cluster

# Ou manuel
ansible-playbook -i inventory.yml cleanup-playbook.yml
```

## 📈 Spécifications techniques

- **Kubernetes** : v1.28.2
- **Runtime** : Containerd v1.7.8
- **CNI** : Flannel v0.22.3
- **Dashboard** : v2.7.0
- **Réseaux** : Pod CIDR 10.244.0.0/16, Service CIDR 10.96.0.0/12

## 🎯 Architecture finale

```
GitLab CI (Synology NAS) → Pipeline → Cluster K8s (3 VMs)
                               ↓
                        Dashboard + Monitoring
                               ↓
                     Prometheus/Grafana (NAS)
```

**Le cluster est maintenant prêt avec interface graphique et monitoring intégré !** 🚀# Pipeline relancé le 10/16/2025 10:54:36
