# 🚀 Jenkins + Terraform + Docker + EKS Deployment Guide

## (Production CI/CD Setup – Pipeline Script from SCM)

This project automates deployment of a **containerized application on AWS EKS** using:

* ✅ **Terraform (Infrastructure as Code)**
* ✅ **AWS EKS (Kubernetes Cluster)**
* ✅ **Docker (Containerization)**
* ✅ **Docker Hub (Image Registry)**
* ✅ **Jenkins CI/CD (Pipeline Script from SCM)**

Repository:

```
https://github.com/faizanmansuri77/easycrud-eks-deployment-by-jenkins.git
```

---

# 📌 Prerequisites

## 🔹 AWS Requirements

* AWS Account (Free Tier not fully supported for EKS)
* IAM User with permissions for:

  * EC2
  * VPC
  * IAM
  * EKS
  * ECR (if used)
* Access Key & Secret Key

---

## 🔹 Required Accounts

* Docker Hub Account
* GitHub Repository

---

# 🟢 STEP 1: Launch EC2 Instance (Ubuntu for Jenkins)

Go to:

```
AWS Console → EC2 → Launch Instance
```

### Select:

* **AMI** → Ubuntu Server 22.04 LTS
* **Instance Type** → c7i-flex.large (Recommended for EKS + Docker builds)
* **Storage** → 30 GB

---

## 🔐 Security Group Ports

| Port | Purpose          |
| ---- | ---------------- |
| 22   | SSH              |
| 8081 | Jenkins          |
| 80   | App LoadBalancer |
| 443  | HTTPS (Optional) |

Launch the instance.

---

## 🔹 Connect to EC2

```bash
ssh -i your-key.pem ubuntu@your-public-ip
```

---

# 🟢 STEP 2: Install Required Software

## ☕ Install Java (Required for Jenkins)

```bash
sudo apt update -y
sudo apt install openjdk-17-jdk -y
```

Verify:

```bash
java -version
```

---

## 💾 Install MySQL Client

```bash
sudo apt install mysql-client -y
```

---

## 🛠 Install Jenkins

```bash
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key

echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt install jenkins -y
```

Start Jenkins:

```bash
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

---
## 🔹 Access Jenkins

Get admin password:

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Open browser:

```
http://<EC2-PUBLIC-IP>:8080
```

Setup Jenkins and Install Suggested Plugins

----

## 🔄 Change Jenkins Default Port (8080 → 8081)

```bash
sudo nano /lib/systemd/system/jenkins.service
```

Change:

```bash
Environment="JENKINS_PORT=8080"
```

To:

```bash
Environment="JENKINS_PORT=8081"
```

Restart:

```bash
sudo systemctl daemon-reload
sudo systemctl restart jenkins
```

Access:

```
http://<EC2-PUBLIC-IP>:8081
```

---

## 🔹 Install & Configure AWS CLI

### Install AWS CLI (Snap)

```bash
sudo snap install aws-cli --classic
```

---

### Configure AWS Credentials

```bash
aws configure
```

Enter:

* **AWS Access Key ID**
* **AWS Secret Access Key**
* **Default region** (e.g., `us-east-1`)
* **Default output format** (`json`)

---


# 🟢 Install Docker

```bash
sudo apt install docker.io -y
sudo systemctl enable docker
sudo systemctl start docker
```

Allow Jenkins to use Docker:

```bash
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
```

---

# 🟢 Install Terraform

```bash
sudo apt install -y gnupg software-properties-common curl

curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o \
  /usr/share/keyrings/hashicorp-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] \
  https://apt.releases.hashicorp.com $(lsb_release -cs) main" | \
  sudo tee /etc/apt/sources.list.d/hashicorp.list

sudo apt update
sudo apt install terraform -y
```

Verify:

```bash
terraform -version
```

---

# 🟢 Install kubectl

```bash
curl -LO "https://dl.k8s.io/release/$(curl -Ls https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

Verify:

```bash
kubectl version --client
```

---

# 🟢 STEP 3: Install Required Jenkins Plugins

Go to:

```
Manage Jenkins → Plugins → Available Plugins
```

Install:

* ✅ Pipeline: Stage View
* ✅ AWS Credentials

Restart Jenkins.

---

# 🟢 STEP 4: Add Credentials in Jenkins

Go to:

```
Manage Jenkins → Credentials → Global → Add Credentials
```

---

## ✅ 1️⃣ AWS Credentials

* Kind → AWS Credentials
* ID → `aws-creds`
* Add Access Key & Secret Key

---

## ✅ 2️⃣ Docker Hub Credentials

* Kind → Username/Password
* ID → `dockerhub-cred`

---

## ✅ 3️⃣ RDS Credentials

* Kind → Username/Password
* ID → `rds-creds`
* Username → `admin`
* Password → `redhat123`

---

# 🟢 STEP 5: Create Jenkins Pipeline (Pipeline Script from SCM)

---

## 🔹 Create New Job

* Click **New Item**
* Name → `easycrud-eks-deployment`
* Select → **Pipeline**
* Click OK

---

## 🔹 Configure Pipeline

Scroll to **Pipeline Section**

```
Definition → Pipeline script from SCM
SCM → Git
```

### Repository URL

```
https://github.com/faizanmansuri77/easycrud-eks-deployment-by-jenkins.git
```

### Branch

```
*/main
```

### Script Path

```
Jenkinsfile
```

Click **Save**

---

# 🟢 STEP 6: Run the Pipeline

Click:

```
Build Now
```

---

# ⚙️ What Happens Automatically

---

## 1️⃣ Jenkins Clones Repository

Clones:

* `backend/`
* `frontend/`
* `terraform/`
* `k8s/`
* `Jenkinsfile`

---

## 2️⃣ Terraform Creates AWS Infrastructure

Terraform provisions:

* VPC
* Subnets
* Internet Gateway
* IAM Roles
* EKS Cluster
* Worker Node Group

---

## 3️⃣ Jenkins Fetches RDS Endpoint

```bash
terraform output rds_endpoint
```

---

## 4️⃣ Jenkins Creates Database & Table

Creates:

* `student_db`
* `admin` user
* `students` table

---

## 5️⃣ Jenkins Updates Backend Configuration

Updates:

```
backend/src/main/resources/application.properties
```

Sets:

* RDS endpoint
* DB port
* Username
* Password
* MariaDB driver

---

## 6️⃣ Jenkins Builds Backend Docker Image

```bash
docker build -t easycrud1-jenkins:backend .
```

---

## 7️⃣ Push Backend Image to Docker Hub

```bash
docker push orionpax77/easycrud1-jenkins:backend
```

---

## 8️⃣ Deploy Backend to EKS & Fetch LB

```bash
kubectl apply -f k8s/backend-deployment.yaml
kubectl get svc backend-svc -o jsonpath={.status.loadBalancer.ingress[0].hostname}
```

---

## 9️⃣ Jenkins Update Frontend .env File

Sets:

```
VITE_API_URL=.*|VITE_API_URL=http://${BACKEND_LB}:8080/api
```

---

## 🔟 Jenkins Builds Frontend Docker Image

```bash
docker build -t easycrud1-jenkins:frontend .
```

---

## 1️⃣1️⃣ Push Backend Image to Docker Hub

```bash
docker push orionpax77/easycrud1-jenkins:frontend
```

---

## 1️⃣2️⃣ Deploy Frontend to EKS 

```bash
kubectl apply -f k8s/frontend-deployment.yaml
kubectl get svc frontend-svc -o jsonpath={.status.loadBalancer.ingress[0].hostname}
```


# ⏳ Expected Deployment Time

| Task                   | Time          |
| ---------------------- | ------------- |
| Terraform EKS Creation | 10–15 minutes |
| Docker Build & Push    | 2–4 minutes   |
| Full Pipeline          | 15–20 minutes |

---

# 🎯 Final Result

After successful pipeline execution:

* ✅ AWS EKS Cluster Created
* ✅ Worker Nodes Running
* ✅ Docker Image Built & Pushed
* ✅ Application Deployed on Kubernetes
* ✅ LoadBalancer Provisioned
* ✅ Fully Automated CI/CD Deployment

---

# 🌐 Access Application

Get External IP:

```bash
kubectl get svc
```

Access:

```
http://<EXTERNAL-LOADBALANCER-DNS>
```

---

# 🛑 Destroy Infrastructure

Navigate to Jenkins workspace:

```bash
cd /var/lib/jenkins/workspace/easycrud-eks-deployment/terraform
terraform destroy --auto-approve
```

Or create a separate destroy pipeline.

---

# 🏁 Conclusion

This project demonstrates:

* ✅ Infrastructure as Code using Terraform
* ✅ Kubernetes Deployment on AWS EKS
* ✅ Docker Containerization
* ✅ Automated CI/CD using Jenkins (Pipeline Script from SCM)
* ✅ Production-grade Cloud Architecture

---
