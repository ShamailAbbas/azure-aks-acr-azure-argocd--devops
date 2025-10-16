# 🚀 Full DevOps Pipeline: AKS + ArgoCD + ACR + Azure DevOps + Node.js App

## 🧭 Overview

This setup builds a **Node.js REST API**, **containerizes it**, **pushes it to Azure Container Registry (ACR)** on each commit (CI), and then **Argo CD** automatically syncs and deploys the new image to **Azure Kubernetes Service (AKS)** (CD).

---

## 📁 Directory Structure

```bash
project-root/
├── infra/
│   ├── main.tf
│   ├── variables.tf
│   └── outputs.tf
├── app/
│   ├── Dockerfile
│   ├── package.json
│   ├── index.js
│   └── k8s/
│       ├── deployment.yaml
│       └── service.yaml
├── azure-pipelines.yml
└── README.md
```

---

## ⚙️ Step 1: Provision AKS + ACR using Terraform

```bash
cd infra
terraform init
terraform apply -auto-approve
```

---

## 🔑 Step 2: Connect to AKS Cluster

```bash
az login
az account set --subscription <your_subscription_id>
az aks get-credentials --resource-group rg-aks-argocd --name aks-argocd-demo
```

Verify:

```bash
kubectl get nodes
```

---

## 🧩 Step 3: Deploy Argo CD on AKS

### Create namespace and install Argo CD:

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### Expose Argo CD server (for external access):

```bash
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```

Wait and get the public IP:

```bash
kubectl get svc -n argocd
```

### Get admin password:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

Login to ArgoCD UI at `http://<EXTERNAL-IP>` with username `admin` and the password above.

---

## 🧠 Step 4: Configure Argo CD to Sync with Repo

In the Argo CD UI:

1. Click **New App**
2. Set:

   - **App name**: node-api
   - **Project**: default
   - **Repo URL**: your GitHub repo (HTTPS)
   - **Path**: `app/k8s`
   - **Cluster URL**: [https://kubernetes.default.svc](https://kubernetes.default.svc)
   - **Namespace**: default

3. Enable **Auto-sync**

Now ArgoCD will deploy whatever’s in `app/k8s/` automatically.

---

## ⚙️ Step 5: Azure DevOps CI Pipeline and ArgoCD

The azure piple code should build the image and push to the acr repo and update the maniufect files to that we have the connect images mentioned in the manifest files.
Then make sure to create an app in argo cd , connect that to the azure repo.Argo CD will detect the change in the GitHub repo (new image tag) and automatically deploy the new version.

---

## ✅ Step 6: Verify Deployment

```bash
kubectl get pods
kubectl get svc node-api-service
```

Open the external IP in your browser — you should see your API response.

---
