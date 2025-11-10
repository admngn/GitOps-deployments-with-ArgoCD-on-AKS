# GitOps-deployments-with-ArgoCD-on-AKS

![Argo CD](https://img.shields.io/badge/GitOps-Argo%20CD-blue?logo=argo)
![Terraform](https://img.shields.io/badge/IaC-Terraform-purple?logo=terraform)
![AKS](https://img.shields.io/badge/Cloud-Azure%20Kubernetes%20Service-0078D4?logo=microsoft-azure)

## 🚀 Objectif

Ce projet montre comment :
- **Provisionner un cluster AKS** (Azure Kubernetes Service) avec **Terraform**  
- **Installer Argo CD** pour la gestion GitOps  
- **Déployer une application** (exemple : *game-2048*) via des manifests Kubernetes versionnés  

L’ensemble du flux GitOps repose sur Argo CD, qui surveille ce dépôt et synchronise automatiquement les ressources déclarées.

---

## 📋 Prérequis

- Compte Azure avec droits *Contributor* sur l’abonnement ou le resource group
- Outils installés en local :
  - [Azure CLI](https://learn.microsoft.com/cli/azure) → `az`
  - [kubectl](https://kubernetes.io/docs/tasks/tools/)
  - [Terraform](https://developer.hashicorp.com/terraform/downloads) ≥ **1.5**
  - [Git](https://git-scm.com/)
  - [Argo CD CLI](https://argo-cd.readthedocs.io/en/stable/cli_installation/) → `argocd`

---
  
## 🧰 Cloner le dépôt

git clone https://github.com/admngn/GitOps-deployments-with-ArgoCD-on-AKS.git
cd GitOps-deployments-with-ArgoCD-on-AKS

---

## Structure du dépôt
```text
GitOps-deployments-with-ArgoCD-on-AKS/
├── README.md
├── argocd/
│   ├── install/                 # Manifests d'installation Argo CD (optionnel si tu appliques tout le dossier)
│   ├── apps/
│   │   └── game-2048.yaml       # (optionnel) Déclaration Argo CD Application
│   └── <autres-yaml>.yaml       # Manifests Argo CD (server, rbac, svc, etc.)
├── infrastructure/
│   └── terraform/
│       ├── main.tf
│       ├── variables.tf
│       ├── outputs.tf
│       └── terraform.tfvars     # tes variables (location, node_count, etc.)
└── manifests/
    └── game-2048/
        ├── kustomization.yaml
        ├── namespace.yaml
        ├── deployment.yaml
        └── service.yaml
```
---

## 🏗️ Étape 1 – Déployement de l’infrastructure AKS avec Terraform

1. Aller dans le répertoire Terraform
```bash
  cd infrastructure/terraform
```
2. Initialiser Terraform
```bash
  terraform init
```
  
3. Configurer les variables
```bash
  Crée un fichier terraform.tfvars :
  prefix      = "gitops-demo"
  location    = "westeurope"
  node_count  = 2
  node_size   = "Standard_DS2_v2"
  
```
4. Appliquer la configuration
```bash
  terraform plan
  terraform apply
```

5. Récupérer le kubeconfig
```bash
  az aks get-credentials -g <RESOURCE_GROUP> -n <AKS_NAME> --overwrite-existing
  kubectl get nodes
```

## ⚙️ Étape 2 — Installer Argo CD

1. Créer le namespace Argo CD
  ```bash
  kubectl create namespace argocd --dry-run=client -o yaml | kubectl apply -f -
  ```
2. Ajoute le repo Helm ArgoCD
  ```bash
  helm repo add argo https://argoproj.github.io/argo-helm
  helm repo update
  ```
3. Installe ArgoCD via Helm
  ```bash
  helm install argocd argo/argo-cd \
    --namespace argocd \
    --create-namespace
  ```
4. Vérifier le déploiement
  ```bash
  kubectl get pods -n argocd
  ```
5. Récupérer le mot de passe admin et accéder à l’interface
  ```bash
  kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d; echo
  
  kubectl -n argocd port-forward svc/argocd-server 8080:443
  ```
  
➜ Accéder à https://localhost:8080
Identifiant : admin
Mot de passe : celui affiché ci-dessus
