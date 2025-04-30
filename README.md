# 2048_app_game / AWS Elastic Kubernetes/ End to End project


## Pre-requisite

**Install kubectl binary with curl on Linux**
```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
kubectl version --client
kubectl get nodes

