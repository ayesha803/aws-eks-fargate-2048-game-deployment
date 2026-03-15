# 2048 Game Deployment on Amazon EKS with Fargate

This project demonstrates how to deploy the **2048 game application** on **Amazon EKS (Elastic Kubernetes Service)** using **AWS Fargate** and expose it through an **AWS Application Load Balancer (ALB)** using the **AWS Load Balancer Controller**.

## Architecture

* Amazon EKS Cluster
* AWS Fargate for serverless compute
* Kubernetes Deployment, Service, and Ingress
* AWS Load Balancer Controller
* Application Load Balancer (ALB)

## Prerequisites

Before starting, ensure the following tools are installed and configured:

* AWS CLI
* eksctl
* kubectl



---

# Step 1: Create EKS Cluster with Fargate

Create an EKS cluster using `eksctl`.

```bash
eksctl create cluster \
  --name demo-cluster \
  --region us-east-1 \
  --fargate
```



---

# Step 2: Update kubeconfig

Configure `kubectl` to connect to the created cluster.

```bash
aws eks update-kubeconfig --name demo-cluster
```

Verify cluster access:

```bash
kubectl get nodes
```

---

# Step 3: Create Fargate Profile

Create a Fargate profile for the namespace where the application will run.

```bash
eksctl create fargateprofile \
    --cluster demo-cluster \
    --region us-east-1 \
    --name alb-sample-app \
    --namespace game-2048
```

---

# Step 4: Deploy the 2048 Application

Deploy the **2048 game application**, service, and ingress into the `game-2048` namespace.

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml
```

This manifest creates:

* Deployment
* Service
* Ingress resource

---

# Step 5: Associate IAM OIDC Provider

The **AWS Load Balancer Controller** requires IAM permissions to create AWS resources such as ALB.

Associate the IAM OIDC provider with the cluster:

```bash
eksctl utils associate-iam-oidc-provider \
  --cluster demo-cluster \
  --approve
```

---

# Step 6: Create IAM Policy

Create the IAM policy required by the AWS Load Balancer Controller.

```bash
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.4/docs/examples/iam-policy.json
```

---

# Step 7: Create IAM Role and Service Account

Create a Kubernetes service account with the required IAM role.

```bash
eksctl create iamserviceaccount \
  --cluster=demo-cluster \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::<ACCOUNT_ID>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
```

---

# Step 8: Install AWS Load Balancer Controller

Install the controller using Helm.

```bash
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system \
  --set clusterName=demo-cluster \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=us-east-1 \
  --set vpcId=<YOUR_VPC_ID>
```

---

# Step 9: Access the Application

Once the ingress is created, an **Application Load Balancer (ALB)** will be provisioned automatically.

Get the ingress details:

```bash
kubectl get ingress -n game-2048
```

Open the **ALB DNS name** in your browser to access the **2048 game**.

---

# Result

The **2048 game application** will be accessible via the **AWS Application Load Balancer URL**.

---

# Technologies Used

* Amazon Elastic Kubernetes Service
* AWS Fargate
* AWS Load Balancer Controller
* Kubernetes
