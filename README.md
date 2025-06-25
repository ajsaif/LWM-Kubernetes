# 🚀 Kubernetes Part 1 – minikube & KOps Installation

This README serves as a **complete hands-on guide** for Kubernetes learning. It includes YAML manifests and command-line usage to help you practice kubernetes related topics.

## 📚 What You’ll Learn

- ✅ How to create a Production Grade Cluster using **kOps**
- ✅ How to create a Local **minikube** Cluster

---

## 🔧 Getting Started with kOps on AWS

[Click me](https://kops.sigs.k8s.io/getting_started/aws/) for Official Documentation of kOps

### 1️⃣ Install Kops for Linux

```bash
curl -Lo kops https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
chmod +x ./kops
sudo mv ./kops /usr/local/bin/
```

### 2️⃣ Install kubectl for Linux

```bash
curl -Lo kubectl https://dl.k8s.io/release/$(curl -s -L https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
```

### 3️⃣ Setup IAM user 

In order to build clusters within AWS we'll create a dedicated IAM user for kops. This user requires API credentials in order to use kops. Create the user, and credentials, using the AWS console.

The kops user will require the following IAM permissions to function properly:

```bash
AmazonEC2FullAccess
AmazonRoute53FullAccess
AmazonS3FullAccess
IAMFullAccess
AmazonVPCFullAccess
AmazonSQSFullAccess
AmazonEventBridgeFullAccess
```

### 4️⃣ Create S3 Bucket 

In order to store the state of your cluster, and the representation of your cluster, we need to create a dedicated S3 bucket for kops to use. This bucket will become the source of truth for our cluster configuration. In this guide we'll call this bucket ==**lwm-kubernetes-bucket**==, but you should add a custom prefix as bucket names need to be unique.

### 5️⃣ Prepare local environment 

You should record the SecretAccessKey and AccessKeyID in the returned JSON output, and then use them below:

```bash
# configure the aws client to use your new IAM user
aws configure           # Use your new access and secret key here
aws iam list-users      # you should see a list of all your IAM users here

# Because "aws configure" doesn't export these vars for kops to use, we export them now
export AWS_ACCESS_KEY_ID=$(aws configure get aws_access_key_id)
export AWS_SECRET_ACCESS_KEY=$(aws configure get aws_secret_access_key)
export NAME=myfirstcluster.k8s.local
export KOPS_STATE_STORE=s3://prefix-example-com-state-store
```

For a gossip-based cluster, make sure the name ends with ==**k8s.local.**==

### 6️⃣ kOps Commands

```bash
kops create cluster --name=${NAME} --cloud=aws --zones=ap-southeast-1a
kops edit cluster --name ${NAME}
kops update cluster --name ${NAME} --yes --admin
kops validate cluster
kops delete cluster --name ${NAME} --yes
```

---

## 🔧 Getting Started with minikube on AWS

[Click me](https://minikube.sigs.k8s.io/docs/start/?arch=%2Flinux%2Fx86-64%2Fstable%2Fbinary+download) for Official Documentation of minikube

### 1️⃣ Installation

Create a Linux EC2 VM with minimum  
- 2 CPUs or more 
- 2GB of free memory 
- 20GB of free disk space

```bash
curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64
```

### 2️⃣ Start your cluster

```bash
minikube start
```

### 3️⃣ Interact with your cluster

```bash
minikube kubectl -- get pod -A
```

You can also make your life easier by adding the following to your shell config

```bash
alias kubectl="minikube kubectl --"
kubectl get pod
```

---

### 📞 Contact Us
**Phone:** [+91 91500 87745](tel:+919150087745)

### 💬 Ask Your Doubts
Join our **Discord Community**  
👉 [Click here to connect](https://discord.gg/N7GBNHBdqw)

### 📺 Explore More Learning
Subscribe to our **YouTube Channel** – *Learn With Mithran*  
🎯 [Watch Now](https://www.youtube.com/@LearnWithMithran)

---

# 🚀 Kubernetes Part 2 – YAML Reference Guide

In Part 2, you will learn how to create **Namespaces, Pods, ReplicaSets, DaemonSets.** Learn about kubectl cheat codes and both imperative and declarative way of deploying kubernetes Objects


## 📚 What You’ll Learn

- ✅ Creating and using **Namespaces**
- ✅ Defining **Pods** with one or more containers
- ✅ Using **ReplicaSets** to maintain pod availability
- ✅ Deploying **DaemonSets** to run a pod on each node

---

## 🔧 How to Use

👉 You can copy all YAMLs below into a file like `main.yaml` and run:

```bash
kubectl apply -f main.yaml
```

### 1️⃣ Namespace: qa

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: qa
```

### 2️⃣ Pod: myfirstpod (Single Container)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myfirstpod
spec:
  containers:
  - name: cont1
    image: httpd
    ports:
    - containerPort: 80
```

### 3️⃣ Pod: mysecondpod (Multi-Container: httpd + jenkins)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mysecondpod
spec:
  containers:
  - name: cont1
    image: httpd
    ports:
    - containerPort: 80
  - name: cont2
    image: jenkins/jenkins
    ports:
    - containerPort: 8080
```

### 4️⃣ Pod: mythirdpod (Multi-Container: httpd + nginx)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mythirdpod
spec:
  containers:
  - name: cont1
    image: httpd
    ports:
    - containerPort: 80
  - name: cont2
    image: nginx
    ports:
    - containerPort: 80
```

### 5️⃣ Pod: myfirstpod in dev Namespace

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myfirstpod
  namespace: dev
spec:
  containers:
  - name: cont1
    image: httpd
    ports:
    - containerPort: 80
```

### 6️⃣ ReplicaSet: lwm-replica (5 Replicas)

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: lwm-replica
spec:
  # modify replicas according to your case
  replicas: 5
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: cont1
        image: httpd
```

### 7️⃣ DaemonSet: lwm-daemon

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: lwm-daemon
spec:
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: cont1
        image: httpd
```
---

### 📞 Contact Us
**Phone:** [+91 91500 87745](tel:+919150087745)

### 💬 Ask Your Doubts
Join our **Discord Community**  
👉 [Click here to connect](https://discord.gg/N7GBNHBdqw)

### 📺 Explore More Learning
Subscribe to our **YouTube Channel** – *Learn With Mithran*  
🎯 [Watch Now](https://www.youtube.com/@LearnWithMithran)

---

# 🚀 Kubernetes Part 3 – YAML Reference Guide

In **Part 3**, you will learn how to manage applications using **Deployments** and expose them to users and other services using **Kubernetes Services**.


## 📚 What You’ll Learn

- ✅ Understand what a Deployment is and how it manages Pods  
- ✅ Create Deployments with different update strategies (RollingUpdate vs Recreate)  
- ✅ Expose Pods and Deployments using various Service types:
    - 🔹 **NodePort** – for accessing Pods externally via a fixed port on the node  
    - 🔹 **ClusterIP** – for internal Pod-to-Pod communication  
    - 🔹 **LoadBalancer** – for external access using cloud provider’s load balancer
- ✅ Demonstrate how **selectors and labels** work in real-world scenarios  
- ✅ Connect Pods internally and externally using appropriate Service types  
- ✅ Expose applications like **Jenkins** via NodePort  
- ✅ Test and validate connections using port mappings

---

## 🔧 How to Use

👉 You can copy all YAMLs below into a file like `main.yaml` and run:

```bash
kubectl apply -f main.yaml
```

### 1️⃣ Basic Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: lwm-deployment
spec:
  replicas: 5
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: cont1
        image: httpd
```

### 2️⃣ Deployment with Recreate Strategy

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: lwm-deployment-re
spec:
  strategy:
    type: Recreate
  replicas: 4
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: cont1
        image: httpd

```

### 3️⃣ Exposing a Pod with NodePort Service

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webserver
  labels:
    app: httpd
spec:
  containers:
  - name: cont1
    image: httpd
    ports:
    - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: my-np-service
spec:
  type: NodePort
  selector:
    app: httpd
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30001
```

### 4️⃣ Exposing Jenkins Pod with NodePort Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-np-service
spec:
  type: NodePort
  selector:
    youtube: lwm
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30002
---
apiVersion: v1
kind: Pod
metadata:
  name: jenkins-pod
  labels:
    youtube: lwm
spec:
  containers:
  - name: cont1
    image: jenkins/jenkins
    ports:
    - containerPort: 8080
```

### 5️⃣ Pod-to-Pod Connection via ClusterIP Service

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod1
  labels:
    colour: black
spec:
  containers:
  - name: cont1
    image: httpd
    ports:
    - containerPort: 80
---
apiVersion: v1
kind: Pod
metadata:
  name: pod2
  labels:
    colour: pink
spec:
  containers:
  - name: cont1
    image: nginx
    ports:
    - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: my-cip-service
spec:
  type: ClusterIP
  selector:
    colour: pink
  ports:
    - port: 9999
      targetPort: 80
```

### 6️⃣ Exposing Deployment with LoadBalancer Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: lb-service
spec:
  type: LoadBalancer
  selector:
    colour: blue
  ports:
    - port: 80
      targetPort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: lwm-deployment
spec:
  replicas: 5
  selector:
    matchLabels:
      colour: blue
  template:
    metadata:
      labels:
        colour: blue
    spec:
      containers:
      - name: cont1
        image: httpd
```

### 7️⃣ Showcasing Selector and Labels with Deployment and Pod

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: lwm-deployment
spec:
  replicas: 5
  selector:
    matchLabels:
      colour: blue
  template:
    metadata:
      labels:
        colour: blue
    spec:
      containers:
      - name: cont1
        image: httpd
---
apiVersion: v1
kind: Pod
metadata:
  name: pod1
  labels:
    colour: blue
spec:
  containers:
  - name: cont1
    image: nginx
    ports:
    - containerPort: 80
```

---

### 📞 Contact Us
**Phone:** [+91 91500 87745](tel:+919150087745)

### 💬 Ask Your Doubts
Join our **Discord Community**  
👉 [Click here to connect](https://discord.gg/N7GBNHBdqw)

### 📺 Explore More Learning
Subscribe to our **YouTube Channel** – *Learn With Mithran*  
🎯 [Watch Now](https://www.youtube.com/@LearnWithMithran)

---

# 🚀 Kubernetes Part 4 – YAML Reference Guide

In **Part 4**, you will learn how to securely manage user access and permissions inside your EKS cluster using:

## 📚 What You’ll Learn

- ✅ `aws-auth` ConfigMap to add IAM users and roles  
- ✅ Role-Based Access Control (RBAC) for namespace-specific and cluster-wide access  
- ✅ Create `Role`, `RoleBinding`, `ClusterRole`, and `ClusterRoleBinding`  
- ✅ Map IAM users to Kubernetes RBAC groups  
- ✅ Use **IRSA (IAM Roles for Service Accounts)** to allow Pods to access AWS services like S3 securely  
- ✅ Hands-on live demo of a Pod accessing S3 using IRSA

---

## 🔧 How to Use

👉 You can copy all YAMLs below into a file like `main.yaml` and run:

```bash
kubectl apply -f main.yaml
```

### 1️⃣ aws-auth ConfigMap – Adding IAM Users

```yaml
apiVersion: v1
data:
  mapRoles: |
    - groups:
      - system:bootstrappers
      - system:nodes
      rolearn: arn:aws:iam::992382429239:role/eks-node-group-role
      username: system:node:{{EC2PrivateDNSName}}
  mapUsers: |
    - userarn: arn:aws:iam::992382429239:user/dev-user
      username: dev-user
      groups:
        - dev-group
    - userarn: arn:aws:iam::992382429239:user/readonly-user
      username: readonly-user
      groups:
        - readonly-group
kind: ConfigMap
metadata:
  name: aws-auth
  namespace: kube-system
```

### 2️⃣ Dev User Role and RoleBinding (Namespace-scoped Access)

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: dev-role
  namespace: dev-namespace
rules:
- apiGroups: ["", "apps", "batch"]
  resources: ["pods", "services", "deployments", "jobs"]
  verbs: ["get", "list", "create", "update", "delete", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dev-binding
  namespace: dev-namespace
subjects:
- kind: Group
  name: dev-group
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: dev-role
  apiGroup: rbac.authorization.k8s.io
```

### 3️⃣ ReadOnly User ClusterRole and ClusterRoleBinding
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: readonly-role
rules:
- apiGroups: ["", "apps", "batch"]
  resources: ["pods", "services", "deployments", "jobs"]
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: readonly-binding
subjects:
- kind: Group
  name: readonly-group
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: readonly-role
  apiGroup: rbac.authorization.k8s.io
```

### 4️⃣ Pod Accessing S3 Using IRSA

#### 🔹 Step 1: Enable OIDC Provider for Your EKS Cluster

```bash
eksctl utils associate-iam-oidc-provider \
  --region ap-southeast-1 \
  --cluster <cluster-name> \
  --approve
```

#### 🔹 Step 2: Create IAM Role with Trust Policy for Pod
#### 🔹 Step 3: Create Kubernetes ServiceAccount

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: s3-access
  namespace: default
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::<account-id>:role/EKS_S3_IRSA_Role
```

#### 🔹 Step 4: Deploy a Pod Using the Service Account

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: s3-reader
spec:
  serviceAccountName: s3-access
  containers:
  - name: awscli
    image: amazonlinux
    command: ["/bin/sh"]
    args: ["-c", "yum install -y aws-cli && aws s3 ls"]
```

#### 🔹 Step 5: Verify Access to S3 from Pod

```bash
2025-06-01 00:00:00 lwm-terraform-bucket
2025-06-01 00:00:00 lwm-kubernetes-bucket
```

---

### 📞 Contact Us
**Phone:** [+91 91500 87745](tel:+919150087745)

### 💬 Ask Your Doubts
Join our **Discord Community**  
👉 [Click here to connect](https://discord.gg/N7GBNHBdqw)

### 📺 Explore More Learning
Subscribe to our **YouTube Channel** – *Learn With Mithran*  
🎯 [Watch Now](https://www.youtube.com/@LearnWithMithran)

---