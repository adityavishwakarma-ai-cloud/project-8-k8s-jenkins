# ☸️ Project 8: Kubernetes CI/CD Pipeline with Jenkins

## 📌 Project Overview

This project demonstrates building a **CI/CD pipeline** to automatically deploy applications to a **Kubernetes cluster** using **Jenkins**.

Key highlights:

* Automates **build, test, and deployment** for containerized applications.
* Deploys **Docker images** to Kubernetes (Minikube, AWS EKS, or GKE).
* Integrates **Jenkins pipelines**, **Docker registry**, and **Kubernetes manifests**.
* Ensures **repeatable, reliable, and scalable deployments**.

---

## 🛠️ Tech Stack / Tools Required

* **Jenkins** (CI/CD automation)
* **Docker & Docker Hub** (Containerization & registry)
* **Kubernetes** (Cluster deployment)
* **kubectl** (K8s CLI)
* **Git & GitHub** (Version control)
* Optional: **Helm** for advanced deployments

---

## 📂 Functional Requirements

* Jenkins pipeline triggers on code commits.
* Build Docker image of the application.
* Push Docker image to Docker Hub (or private registry).
* Apply Kubernetes manifests to deploy/update the app in the cluster.
* Notify on successful deployment.

---

## 📂 Non-Functional Requirements

* Pipeline should be **automated and idempotent**.
* Kubernetes deployment must be **scalable and fault-tolerant**.
* Ensure **secure access** to Docker registry and cluster.
* Provide **logging and monitoring** for deployments.

---

## 🔄 Project Flow

```
1. Developer pushes code to GitHub repository
       |
       v
2. Jenkins detects commit (Webhook or Polling)
       |
       v
3. Jenkins Pipeline stages:
       ├─ Build Docker image
       ├─ Run unit tests
       ├─ Push image to Docker Hub
       └─ Apply Kubernetes manifests
               |
               v
4. Kubernetes deploys/updates pods and services
       |
       v
5. Application accessible via ClusterIP / LoadBalancer
       |
       v
6. Jenkins reports success/failure
```

---

## 📂 Folder Structure

```
project-8-k8s-jenkins/
│-- README.md
│-- Jenkinsfile                 # Declarative pipeline
│-- k8s-manifests/
│   ├── deployment.yaml
│   └── service.yaml
│-- app/                        # Sample application (Node.js/Django)
│   ├── Dockerfile
│   └── src/
│-- .gitignore
```

---

## 📝 Sample Jenkinsfile

```groovy
pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = credentials('dockerhub-credentials')
        IMAGE_NAME = "your-dockerhub-username/project8-app"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/<your-username>/project8-app.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:${BUILD_NUMBER} .'
            }
        }
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh 'echo $PASSWORD | docker login -u $USERNAME --password-stdin'
                    sh 'docker push $IMAGE_NAME:${BUILD_NUMBER}'
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl apply -f k8s-manifests/deployment.yaml'
                sh 'kubectl apply -f k8s-manifests/service.yaml'
            }
        }
    }
    post {
        success { echo 'Deployment Successful!' }
        failure { echo 'Deployment Failed!' }
    }
}
```

---

## 📂 Sample Kubernetes Deployment – `deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: project8-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: project8-app
  template:
    metadata:
      labels:
        app: project8-app
    spec:
      containers:
      - name: project8-app
        image: your-dockerhub-username/project8-app:latest
        ports:
        - containerPort: 3000
```

## 📂 Sample Kubernetes Service – `service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: project8-app-service
spec:
  type: LoadBalancer
  selector:
    app: project8-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
```

---

## ⚡ Deployment Steps

1. **Start Kubernetes cluster** (Minikube or cloud).

```bash
minikube start
```

2. **Build Docker image locally (optional)**

```bash
docker build -t your-dockerhub-username/project8-app .
```

3. **Push Docker image to registry**

```bash
docker push your-dockerhub-username/project8-app
```

4. **Apply Kubernetes manifests**

```bash
kubectl apply -f k8s-manifests/
```

5. **Access service**

```bash
kubectl get svc project8-app-service
```

6. **Configure Jenkins** with credentials and pipeline.

---

## 📝 Resume Highlights

**Objective**
Built a **CI/CD pipeline** to automate containerized application deployment on Kubernetes.

**Key Achievements**

* Automated **build, test, and deployment** using Jenkins.
* Integrated **Docker, Kubernetes, and GitHub** for end-to-end pipeline.
* Ensured **scalability, fault-tolerance, and repeatable deployments**.

**Impact**
✅ Reduced manual deployment errors.
✅ Enabled fast, automated, and repeatable releases.
✅ Demonstrated DevOps best practices with Kubernetes and Jenkins.

---

## 🔗 Repository Link

```md
https://github.com/<your-username>/project-8-k8s-jenkins
```

