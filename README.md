# 2048_app_game / AWS Elastic Kubernetes/ End to End project


## Pre-requisite

### Install kubectl binary with curl on Linux**

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
kubectl version --client
kubectl get nodes
```

### Install eksctl

```bash
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_Linux_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
```

### Install AWS CLI
```bash
curl "https://s3.amazonaws.com/aws-cli/awscli-bundle-1.16.312.zip" -o "awscli-bundle.zip"
unzip awscli-bundle.zip
sudo ./awscli-bundle/install -i /usr/local/aws -b /usr/local/bin/aws
aws --version
```

Let's configure AWS CLI
```bash
aws configure
```

### Let's begin with deploying the Application
Create a cluster using AWS CLI

```cli
eksctl create cluster --name demo-cluster --region us-east-1 --fargate
```

```cli
aws eks update-kubeconfig --name demo-cluster --region us-east-1
```

Create Fargate profile
```cli
eksctl create fargateprofile \
    --cluster demo-cluster \
    --region us-east-1 \
    --name alb-sample-app \
    --namespace game-2048
```

Deploy the deployment, service and Ingress
```cli
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml
```
