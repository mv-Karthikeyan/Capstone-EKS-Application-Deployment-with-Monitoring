# 🚀 End-to-End MERN Application Deployment on Amazon EKS

This repository contains a complete Kubernetes deployment setup for a MERN (MongoDB, Express.js, React, Node.js) application using:

## 🛠️ Key Features & Tools

- 🧠 **EKS** → Kubernetes cluster provisioning and management  
- 📦 **Helm** → Templated, reusable Kubernetes manifests for simplified deployment  
- 🐳 **Docker** → Containerization of backend, frontend, and database services  
- ⚙️ **Jenkins** (optional) → CI/CD pipeline automation for seamless deployment  
- 🌍 **Terraform & AWS CLI** → Infrastructure as Code (IaC) for AWS resource provisioning  
- 🔒 **ECR** → Secure container image registry for application images
- 📊 **Prometheus** → Metrics collection and alerting for application & cluster monitoring  
- 📈 **Grafana** → Interactive dashboards for real-time visualization and insights   




### Step 1: IAM Configuration
- Create a user `eks-admin` with `AdministratorAccess`.
- Generate Security Credentials: Access Key and Secret Access Key.

### Step 2: EC2 Setup
- Launch an Ubuntu instance in your favourite region (eg. region `us-west-2`).
- SSH into the instance from your local machine.

### Step 3: Install AWS CLI v2
``` shell
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install -i /usr/local/aws-cli -b /usr/local/bin --update
aws configure
```

### Step 4: Install Docker
``` shell
sudo apt-get update
sudo apt install docker.io
docker ps
sudo chown $USER /var/run/docker.sock
```

### Step 5: Build Docker images
``` shell
docker build -t wanderlust-backend
```
 **After the build completes, tag your image so you can push the image to this repository:**
``` shell
docker tag wanderlust-backend:latest public.ecr.aws/x4m1c1q0/wanderlust-backend:latest
```
**Run the following command to push this image to your newly created AWS repository**
``` shell
docker push public.ecr.aws/x4m1c1q0/wanderlust-backend:latest
```

### Step 6: Install kubectl
``` shell
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short --client
```

### Step 7: Install eksctl
``` shell
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```

### Step 8: Setup EKS Cluster
``` shell
eksctl create cluster \
--name three-tier-cluster \
--region ap-south-1 \
--version 1.31 \
--node-type t3.small \
--nodes 2 \
--managed \
--timeout 60m
aws eks update-kubeconfig --region ap-south-1 --name three-tier-cluster
kubectl get nodes
```
<img width="941" height="459" alt="image (2)" src="https://github.com/user-attachments/assets/4570952b-5cf5-472b-8dcf-9d1e403bb426" />

<img width="800" height="166" alt="image (3)" src="https://github.com/user-attachments/assets/77124c0d-6501-41b4-81e8-159ce50fb7bc" />


### Step 9: Run Manifests
``` shell
kubectl create namespace three-tier
kubectl config set-context --current --namespace three-tier
kubectl apply -f .
kubectl delete -f .
```
<img width="950" height="119" alt="image" src="https://github.com/user-attachments/assets/47929f62-8473-41ad-b8d2-a4c9a2bbc611" />

<img width="950" height="357" alt="image" src="https://github.com/user-attachments/assets/9c3c4d0d-41ff-4fe0-ab2e-b3287231d673" />

### Step 10: Install prometheus and Grafana
# Monitoring Setup on EKS using Helm

This guide explains how to install Prometheus and Grafana on an EKS cluster using Helm.

---

# Prerequisites

- EKS Cluster running
- kubectl configured
- Helm installed

Verify cluster access:

```bash
kubectl get nodes
```

---

# 1. Add Prometheus Helm Repository

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

---

# 2. Update Helm Repositories

```bash
helm repo update
```

---

# 3. Create Monitoring Namespace

```bash
kubectl create namespace monitoring
```

---

# 4. Install Prometheus

```bash
helm install prometheus prometheus-community/prometheus -n monitoring
```

---

# 5. Expose Prometheus using LoadBalancer

```bash
helm upgrade prometheus prometheus-community/prometheus \
  -n monitoring \
  --set server.service.type=LoadBalancer \
  --set server.persistentVolume.enabled=false \
  --set alertmanager.persistentVolume.enabled=false
```

This exposes Prometheus externally through an AWS LoadBalancer.

---

# 6. Add Grafana Helm Repository

```bash
helm repo add grafana https://grafana.github.io/helm-charts
```

---

# 7. Update Helm Repositories Again

```bash
helm repo update
```

---

# 8. Install Grafana

```bash
helm install grafana grafana/grafana \
  --namespace monitoring \
  --set service.type=LoadBalancer \
  --set persistence.enabled=false \
  --set adminUser=admin \
  --set adminPassword=admin123
```

---

# 9. Verify Monitoring Pods

```bash
kubectl get pods -n monitoring
```

Wait until all pods are in `Running` state.

Example:

```text
NAME                                                     READY   STATUS    RESTARTS   AGE
grafana-xxxxxxxxxx-xxxxx                                 1/1     Running   0          2m
prometheus-server-xxxxxxxxxx-xxxxx                       2/2     Running   0          5m
```

---

# 10. Get External LoadBalancer URLs

```bash
kubectl get svc -n monitoring
```

Look for the `EXTERNAL-IP` column.

Example:

```text
NAME                TYPE           CLUSTER-IP       EXTERNAL-IP
grafana             LoadBalancer   10.100.135.2    a1b2c3.amazonaws.com
prometheus-server   LoadBalancer   10.100.6.109    d4e5f6.amazonaws.com
```

---

# 11. Access Monitoring Dashboards

## Prometheus

```text
http://<prometheus-external-ip>
```

## Grafana

```text
http://<grafana-external-ip>
```

### Grafana Login Credentials

```text
Username: admin
Password: admin123
```

---

# 12. If EXTERNAL-IP Shows Pending

AWS LoadBalancer creation may take a few minutes.

Recheck services:

```bash
kubectl get svc -n monitoring
```

---

# 13. Useful Verification Commands

## Check Helm Releases

```bash
helm list -n monitoring
```

## Check Services

```bash
kubectl get svc -n monitoring
```

## Check Pods

```bash
kubectl get pods -n monitoring
```

---

# Monitoring Architecture

```text
EKS Cluster
   ↓
Application Pods
   ↓
Prometheus collects metrics
   ↓
Grafana visualizes metrics
```
## ⚙️ Jenkins configuration

---

## ✅ Prerequisites

- 🐧 Ubuntu 24.04 system
- 🔑 `sudo` privileges
- ☕ Java (OpenJDK 11 or 17 — recommended: 17)

---

## 🔄 Post-Installation Recommendations

✅ Install the following plugins during setup:

- Pipeline
- Docker Pipeline
- Git
- SSH Agent
- Credentials Binding
- Kubernetes CLI
- Helm

---

## 🛠️ Installation Steps

### 1. 🔄 Update System Packages

```bash
sudo apt update && sudo apt upgrade -y
```

### 2. ☕ Install Java (OpenJDK 17)

```bash
sudo apt install -y openjdk-17-jdk
```

🔍 Verify Java installation:

```bash
java -version
```

### 3. 📦 Add Jenkins Repository and Key

```bash
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key

echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" | \
  sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
```

### 4. 🧰 Install Jenkins

```bash
sudo apt update
sudo apt install -y jenkins
```

### 5. 🚀 Start and Enable Jenkins

```bash
sudo systemctl enable jenkins
sudo systemctl start jenkins
```

Check Jenkins service status:

```bash
sudo systemctl status jenkins
```

### 6. 🔥 Open Firewall (if enabled)

```bash
sudo ufw allow 8080
sudo ufw allow OpenSSH
sudo ufw enable
```

### 7. 🌐 Access Jenkins

Open your browser and go to: `http://your_server_ip:8080`

🔑 Retrieve the initial admin password:

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Follow the setup wizard to:

- Create your first admin user 👤
- Install recommended plugins 🔌
- Configure Jenkins as needed ⚙️

### 🔧 Optional: Change Jenkins Port

Edit the Jenkins config file:

```bash
sudo nano /etc/default/jenkins
```

Change the line:

```
HTTP_PORT=8080
```

Restart Jenkins:

```bash
sudo systemctl restart jenkins
```

---

### Pipeline setup
1. Create Jenkinsfile inside your project directory.
2. Configure Jenkins Credentials Go to Manage Jenkins → Credentials →  AWS Access Key ID  & Secret Access Key (for EKS access) These credentials will be used in the pipeline securely.
3. Set up Git integration for Jenkins to trigger builds based on code changes.
4. Create jenkins pipeline.

```
pipeline {
  agent any
  options {
    timestamps()
  }
  environment {
    AWS_REGION        = 'us-west-2'
    CLUSTER_NAME      = 'three-tier-cluster'
    NAMESPACE         = 'three-tier'
    ECR_REPO_FRONTEND = 'wanderlust-frontend'
    ECR_REPO_BACKEND  = 'wanderlust-backend'
    ECR_REGISTRY      = "863541429677.dkr.ecr.us-west-2.amazonaws.com" // <-- replace with your AWS account ID
  }
  stages {
    stage('Init AWS + Vars') {
      steps {
        script {
          withAWS(region: "${AWS_REGION}", credentials: 'aws-eks-creds') {
            echo "AWS credentials and region set"
          }
        }
      }
    }
    stage('Checkout Code') {
      steps {
        git branch: 'devops', url: 'https://github.com/vipulsaw/Capstone-EKS-Application-Deployment-with-Monitoring.git'
      }
    }
    stage('Configure kubeconfig') {
      steps {
        script {
          withAWS(region: "${AWS_REGION}", credentials: 'aws-eks-creds') {
            sh """
              set -e
              aws eks --region ${AWS_REGION} update-kubeconfig --name ${CLUSTER_NAME}
            """
          }
        }
      }
    }
    stage('Ensure ECR repos') {
      steps {
        script {
          withAWS(region: "${AWS_REGION}", credentials: 'aws-eks-creds') {
            sh """
              set -e
              aws ecr describe-repositories --repository-names ${ECR_REPO_FRONTEND} >/dev/null 2>&1 || \
                aws ecr create-repository --repository-name ${ECR_REPO_FRONTEND}
              aws ecr describe-repositories --repository-names ${ECR_REPO_BACKEND} >/dev/null 2>&1 || \
                aws ecr create-repository --repository-name ${ECR_REPO_BACKEND}
            """
          }
        }
      }
    }
    stage('Login to ECR') {
      steps {
        script {
          withAWS(region: "${AWS_REGION}", credentials: 'aws-eks-creds') {
            sh """
              aws ecr get-login-password --region ${AWS_REGION} | \
              docker login --username AWS --password-stdin ${ECR_REGISTRY}
            """
          }
        }
      }
    }
    stage('Build Frontend Image') {
      steps {
        sh """
          cd frontend
          docker build -t ${ECR_REGISTRY}/${ECR_REPO_FRONTEND}:latest .
        """
      }
    }
    stage('Build Backend Image') {
      steps {
        sh """
          cd backend
          docker build -t ${ECR_REGISTRY}/${ECR_REPO_BACKEND}:latest .
        """
      }
    }
    stage('Push Images to ECR') {
      steps {
        script {
          withAWS(region: "${AWS_REGION}", credentials: 'aws-eks-creds') {
            sh """
              docker push ${ECR_REGISTRY}/${ECR_REPO_FRONTEND}:latest
              docker push ${ECR_REGISTRY}/${ECR_REPO_BACKEND}:latest
            """
          }
        }
      }
    }
    stage('Apply Manifests to EKS') {
      steps {
        script {
          withAWS(region: "${AWS_REGION}", credentials: 'aws-eks-creds') {
            sh """
              echo "Applying backend deployment and service..."
              kubectl apply -f k8s-Manifests/backend/deployment.yaml -n ${NAMESPACE}
              kubectl apply -f k8s-Manifests/backend/service.yaml -n ${NAMESPACE}
              echo "Waiting for backend pods to be ready..."
              kubectl rollout status deployment/api -n ${NAMESPACE} --timeout=180s
              echo "Applying frontend deployment and service..."
              kubectl apply -f k8s-Manifests/frontend/deployment.yaml -n ${NAMESPACE}
              kubectl apply -f k8s-Manifests/frontend/service.yaml -n ${NAMESPACE}
              echo "Waiting for frontend pods to be ready..."
              kubectl rollout status deployment/frontend -n ${NAMESPACE} --timeout=180s
            """
          }
        }
      }
    }
  }
}
```

<img width="1079" height="652" alt="image" src="https://github.com/user-attachments/assets/fcb50ad1-c83d-4d47-a3e8-4089b0627b87" />

<img width="1080" height="655" alt="image" src="https://github.com/user-attachments/assets/4f2b87cd-fba0-4fac-975f-6f2d3ad5588b" />

<img width="1080" height="661" alt="wanderlust-landing" src="https://github.com/user-attachments/assets/1680dbec-9101-44db-b2b3-3bb369d92482" />

<img width="1080" height="662" alt="Create_Blog" src="https://github.com/user-attachments/assets/6abf4b60-49cd-40ab-b97b-c16324088da0" />

<img width="1079" height="665" alt="blog-added" src="https://github.com/user-attachments/assets/33676805-82f3-466e-8e8a-87ad293097d5" />

<img width="955" height="500" alt="image" src="https://github.com/user-attachments/assets/49708fe4-f474-45f8-bb06-9c1e74e05c50" />
<img width="1061" height="659" alt="image" src="https://github.com/user-attachments/assets/bee9255c-e26a-4f0d-9fec-d93cf6948ccc" />

<img width="1054" height="647" alt="image" src="https://github.com/user-attachments/assets/7bac6347-afe2-4295-ab07-bd26529db480" />
<img width="1058" height="650" alt="image" src="https://github.com/user-attachments/assets/bb9b060d-4fed-4d58-9256-0f6486219712" />
<img width="1063" height="654" alt="image" src="https://github.com/user-attachments/assets/0bfa96d5-cf12-42b4-9551-fb48fa8ad7da" />



### Cleanup
- To delete the EKS cluster:
``` shell
eksctl delete cluster --name three-tier-cluster --region us-west-2
```
- To clean up rest of the stuff and not incure any cost
```
Stop or Terminate the EC2 instance created in step 2.
Delete the Load Balancer created in step 9 and 10.
Go to EC2 console, access security group section and delete security groups created in previous steps
```

---
Happy Learning! 🚀👨‍💻👩‍💻


**For Monitoring Promotheus and Grafana**
1.snap install helm --classic
2.Create a namespace for monitoring
3.helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
4.helm repo update
5.helm install prometheus prometheus-community/prometheus\ -n monitoring\ --set server.service.type=loadbalancer\ --set

6.we have to give 3000 port inbound rule in security group with ---> VPN IP/MyIP
7.go to server and give command "kubectl get all -n monitoring"
8.After getting load balancers of promotheus & grafana 
9.for grafana url:3000
10.IN Grafana ---> go for connections ---> data sources ---> add prometheus
11.give prometheus server URL 
12.go to search for " kubectl grafana dashboard"  ,Take the ID number
13.Go to grafana dashboard ---> import dashboard ---> select prometheus in different boxes.



**###Ticket dash board SLA working**

**Key components and functions:**
1. Automated metric tracking: The system automatically tracks key performance indicators (KPIs) like "First Response Time" and "Resolution Time" based on the SLA policies applied to tickets.
2. Configurable SLA policies: SLAs are defined with specific targets (e.g., respond to a high-priority ticket within 30 minutes) and can be triggered by various factors, such as the ticket's priority level, associated account, or ticket type.
3. Real-time visualization: Dashboards present this data visually through charts and graphs, showing the ratio of achieved versus breached tickets over a period, and highlighting which metrics are underperforming.
4. Alerting and proactive monitoring: Many systems will flag tickets that are approaching a breach, allowing support staff to intervene before the deadline is missed.
5. Performance analysis: Managers can use the dashboard to analyze the root causes of SLA breaches, such as high ticket volume, specific types of issues, or team workload.
Reporting and insights: Dashboards provide a comprehensive view of SLA adherence, which can be filtered by various ticket properties to give deeper insights into team, agent, or customer-specific performance. 



**Velero for backup**

for reference browse the **Run velero in AWS** in google 

velero is need to be install in local/ec2 instance which is Client and another is manager which is to EKS for making backups 


Firstly create the service user in AWS IAM account for to use the access and secret keys to EKS backups 
create the AWS user for velero and attache policy and create the access and secret keys 

{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeVolumes",
                "ec2:DescribeSnapshots",
                "ec2:CreateTags",
                "ec2:CreateVolume",
                "ec2:CreateSnapshot",
                "ec2:DeleteSnapshot"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:DeleteObject",
                "s3:PutObject",
                "s3:AbortMultipartUpload",
                "s3:ListMultipartUploadParts"
            ],
            "Resource": [
                "arn:aws:s3:::velero-backup-wanderlust/*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::velero-backup-wanderlust"
            ]
        }
    ]
}


create the S3 buckets and take arn of bucket and give in the policy , and also make S3 bucket public and enable versioning 

Create a file in the ec2 instance/local for the access keys of velero 

vi velero-credentials  

[default]
aws_access_key_id=<AWS_ACCESS_KEY_ID>
aws_secret_access_key=<AWS_SECRET_ACCESS_KEY>


Install the velero in the manager means EKS 

velero install \
    --provider aws \
    --plugins velero/velero-plugin-for-aws:v1.9.0 \
    --bucket velero-backup-wanderlust \
    --secret-file ./credentials-velero \
    --backup-location-config region=ap-south-1 \
    --snapshot-location-config region=ap-south-1

We can see after it installing in Namespaces 

kubectl get ns 

--> make kubectl -n velero get all

let wait for the pod is running 

As per the project is running , now we need to take backup to S3 bucket 

kubectl backup create three-tier-backup --include-namespaces three-tier

here 
three-tier-backup ----> name to S3 bucket to identify
three-tier ----> name of the application deployment 

we can see the backups in S3 bucket 

now try to delete the application namespane 

kubectl delete ns three-tier ---> application will delete 

we can restore it in to the same EKS cluster 

give command

velero get backups

we can see the all backups

velero restore create --from-backup three-tier

we can see the three-tier namespace in the namespaces 






