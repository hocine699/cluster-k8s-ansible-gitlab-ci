# 🚀 Préparation pour GitLab - Projet Kubernetes

## 📋 Configuration complète pour votre GitLab sur Synology

### 1. Créer le nouveau projet dans GitLab
1. Allez sur votre GitLab Synology
2. Cliquez sur **"New Project"**
3. Nom du projet : `kubernetes-cluster-automation`
4. Visibilité : **Private**
5. **Créer le projet**

### 2. Configuration Git locale
```bash
cd "c:\Users\hocin\Documents\J'apprends-l'IA\ansible-k8s-cluster"

# Initialiser Git
git init

# Ajouter le remote GitLab (remplacez par votre URL)
git remote add origin http://[IP-SYNOLOGY]:[PORT]/hocine/kubernetes-cluster-automation.git

# Premier commit et push
git add .
git commit -m "Initial: Kubernetes cluster automation avec mon-runner-devops"
git push -u origin main
```

### 3. Variables GitLab CI/CD
Dans GitLab : **Settings > CI/CD > Variables**

| Variable | Value | Protected | Masked |
|----------|-------|-----------|---------|
| `K8S_MASTER_IP` | `192.168.1.72` | ✓ | ✓ |
| `K8S_WORKER1_IP` | `192.168.1.11` | ✓ | ✓ |
| `K8S_WORKER2_IP` | `192.168.1.12` | ✓ | ✓ |
| `SSH_PRIVATE_KEY` | [contenu complet de votre clé SSH] | ✓ | ✓ |

### 4. Préparation des VMs
```bash
# Copier votre clé SSH sur chaque VM
ssh-copy-id hocine@192.168.1.72
ssh-copy-id hocine@192.168.1.11
ssh-copy-id hocine@192.168.1.12

# Test de connectivité
ssh hocine@192.168.1.72 "uptime"
```

## ✅ Configuration actuelle

Votre pipeline est configurée pour :
- ✅ **Image Docker** : `mon-runner-devops:latest`
- ✅ **Runner tag** : `synology`
- ✅ **Utilisateur** : `hocine`
- ✅ **Master** : `192.168.1.72`
- ✅ **Workers** : `192.168.1.11`, `192.168.1.12`

## 🚀 Après le push

1. La pipeline se déclenchera automatiquement
2. Utilisera votre image `mon-runner-devops` avec toutes les dépendances
3. Déploiera le cluster Kubernetes
4. Générera les artefacts avec accès dashboard et monitoring

## 🎯 Résultat final

Après déploiement automatique :
- **Dashboard K8s** : `https://192.168.1.72:30443`
- **Monitoring** : Endpoints pour votre Prometheus
- **kubeconfig** : Dans les artefacts GitLab