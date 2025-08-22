## 1️⃣ Folder Structure

```
project-8-k8s-jenkins/
│-- README.md
│-- Jenkinsfile                  # CI/CD pipeline
│-- k8s-manifests/
│   ├── deployment.yaml          # Kubernetes deployment
│   └── service.yaml             # Kubernetes service
│-- app/                         # Sample Node.js application
│   ├── Dockerfile
│   └── src/
│       └── index.js
│-- .gitignore
```

---

## 2️⃣ Sample Node.js App – `app/src/index.js`

```javascript
const express = require('express');
const app = express();
const PORT = process.env.PORT || 3000;

app.get('/', (req, res) => {
    res.send('Hello from Project 8 - Kubernetes CI/CD Pipeline!');
});

app.listen(PORT, () => {
    console.log(`Server running on port ${PORT}`);
});
```

---

## 3️⃣ Dockerfile – `app/Dockerfile`

```dockerfile
# Use Node.js LTS image
FROM node:18-alpine

# Create app directory
WORKDIR /usr/src/app

# Copy package.json and install dependencies
COPY package*.json ./
RUN npm install

# Copy app source
COPY ./src ./src

# Expose port
EXPOSE 3000

# Start app
CMD ["node", "src/index.js"]
```

> Note: Add a simple `package.json` for the app:

```json
{
  "name": "project8-app",
  "version": "1.0.0",
  "main": "src/index.js",
  "dependencies": {
    "express": "^4.18.2"
  },
  "scripts": {
    "start": "node src/index.js"
  }
}
```

---

## 4️⃣ Kubernetes Manifests – `k8s-manifests/`

### a) Deployment – `deployment.yaml`

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

### b) Service – `service.yaml`

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

## 5️⃣ Jenkins Pipeline – `Jenkinsfile`

```groovy
pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = credentials('dockerhub-credentials')
        IMAGE_NAME = "your-dockerhub-username/project8-app"
    }

    stages {
        stage('Checkout') {
            steps { git 'https://github.com/<your-username>/project-8-k8s-jenkins.git' }
        }

        stage('Build Docker Image') {
            steps { sh 'docker build -t $IMAGE_NAME:${BUILD_NUMBER} ./app' }
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

## 6️⃣ Deployment Steps

1. **Start Kubernetes cluster** (Minikube or cloud).

```bash
minikube start
```

2. **Build & Push Docker image**

```bash
docker build -t your-dockerhub-username/project8-app ./app
docker push your-dockerhub-username/project8-app
```

3. **Apply Kubernetes manifests**

```bash
kubectl apply -f k8s-manifests/
```

4. **Configure Jenkins**

* Add **Docker Hub credentials**.
* Point pipeline to this repo.
* Trigger pipeline to automate build, push, and deployment.

5. **Verify deployment**

```bash
kubectl get pods
kubectl get svc project8-app-service
```

