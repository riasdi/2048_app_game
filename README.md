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

Lets check if the pods, services are running

```cli
kubectl get pods -n game-2048
kubectl get svc -n game-2048
kubectl get ingress -n game-2048
```
IAM OIDC provider
```cli
eksctl utils associate-iam-oidc-provider --cluster demo-cluster --approve
```
Download IAM policy
```cli
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
```

Create IAM Policy
```cli
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json
```

Create IAM Role
```cli
eksctl create iamserviceaccount \
  --cluster=demo-cluster \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
```

## Deploying ALB controller
Add helm repo
```cli
helm repo add eks https://aws.github.io/eks-charts
```
Update the repo
```cli
helm repo update eks
```
Install ALB Controller
```cli
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \            
  -n kube-system \
  --set clusterName=demo-cluster \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=us-east-1 \
  --set vpcId=<your-vpc-id>
```

Verify if the deployment is running
```cli
kubectl get deployment -n kube-system aws-load-balancer-controller
```
Check for ingress and open the address in new-tab
```cli
kubectl get ingress -n game-2048
```
