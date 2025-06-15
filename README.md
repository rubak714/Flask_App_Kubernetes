# 🔗 DevOps_Project_02: Kubernetes Integration with Flask + Jenkins + Docker

This guide walks you through extending your CI/CD Flask project by deploying it to **Kubernetes (K8s)** using **Minikube**. You'll learn what Kubernetes is, how it works, and how to automate deployments from Jenkins.

---

## 🔗 What is Kubernetes?

**Kubernetes** is a container orchestration platform that automates deployment, scaling, and management of containerized applications.

### 🔗 Key Benefits

* Self-healing: auto-restarts failed containers
* Load balancing: distributes traffic
* Auto-scaling: adjusts replicas based on load
* Declarative configuration via YAML

---

## 🔗 Core Components of Kubernetes

| Component            | Description                                       |
| -------------------- | ------------------------------------------------- |
| **Pod**              | Smallest unit, contains one or more containers    |
| **Node**             | A VM or physical machine running Kubernetes       |
| **Cluster**          | A group of nodes managed by a control plane       |
| **Deployment**       | Declarative way to define and manage Pods         |
| **Service**          | Exposes Pods to the outside world                 |
| **ConfigMap/Secret** | Injects configuration and sensitive data securely |

---

## 🔗 How Kubernetes Works: Visual Overview

```mermaid
flowchart TD
    User -->|kubectl apply| API[API Server]
    API --> Scheduler
    Scheduler --> Node1
    Scheduler --> Node2
    Node1 -->|Pods| Container1
    Node2 -->|Pods| Container2
    Service -->|Routes| Container1 & Container2
```

---

## 🔗 Step-by-Step: Add Kubernetes to Your CI/CD Project

### 🔗 Prerequisites

* Docker
* Jenkins
* Flask App
* Installed:

  * `kubectl`
  * `minikube`

---

## 🔗 Project Folder Structure (Updated)

```
flask-cicd-pipeline/
├── app/
│   ├── app.py
│   └── requirements.txt
├── Jenkinsfile
├── Dockerfile
├── k8s/
│   ├── deployment.yaml
│   └── service.yaml
├── README.md
└── .gitignore
```

---

## 🔗 Kubernetes Manifests

### `k8s/deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flask-app-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: flask-app
  template:
    metadata:
      labels:
        app: flask-app
    spec:
      containers:
      - name: flask-app
        image: flask-cicd-app
        ports:
        - containerPort: 5000
```

### `k8s/service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: flask-service
spec:
  type: NodePort
  selector:
    app: flask-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5000
      nodePort: 30036
```

---

## 🔗 Run Minikube & Build Docker Image

```bash
# Start minikube
minikube start --driver=docker

# Point terminal to minikube's Docker
eval $(minikube docker-env)

# Build Docker image
docker build -t flask-cicd-app .
```

---

## 🔗 Deploy Flask App to Kubernetes

```bash
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
```

```bash
# Check deployments
kubectl get deployments
kubectl get pods
kubectl get svc

# Access the app
minikube service flask-service
```

---

## 🔗 Update Jenkins Pipeline for Kubernetes

### 🔗 Add Stage to `Jenkinsfile`

```groovy
stage('Deploy to Kubernetes') {
  steps {
    script {
      sh 'kubectl apply -f k8s/deployment.yaml'
      sh 'kubectl apply -f k8s/service.yaml'
    }
  }
}
```

> Jenkins must have `kubectl` access.

---

## 🔗 Visual Workflow Interpretation

This section provides a breakdown of the visual workflow represented in the Mermaid diagram from the project. Each interaction between components is explained to clarify how the CI/CD pipeline works in practice.

---

## 🔗 Mermaid Diagram

```mermaid
sequenceDiagram
  participant Dev as Developer
  participant Git as GitHub
  participant Jenkins as Jenkins
  participant Docker as Docker
  participant K8s as Kubernetes
  participant User as End User

  Dev->>Git: Push code
  Git->>Jenkins: Trigger Pipeline
  Jenkins->>Docker: Build Image
  Jenkins->>K8s: Deploy YAML
  K8s->>User: Expose via Service
```
---

## 🔗 Step-by-Step Interpretation

### 1. `Dev ->> Git: Push code`

* **What it means:** The developer writes or updates the application code (for example, a Flask app) and pushes it to a GitHub repository.
* This step uploads the latest code to GitHub, which is the version control system.
* **Why it's important:** This is the starting point of the automation process. Pushing code tells the system it's time to start the pipeline.

### 2. `Git ->> Jenkins: Trigger Pipeline`

* **What it means:** GitHub lets Jenkins know that new code has been pushed. This is usually done through a webhook or Jenkins checking GitHub regularly.
* This is how Jenkins knows it should begin running the automated steps.
* **Why it's important:** It helps make sure every code change is automatically tested, built, and deployed without the developer needing to do anything extra.

### 3. `Jenkins ->> Docker: Build Image`

* **What it means:** Jenkins runs a step in its pipeline to create a Docker image from the updated code.
* In simple terms, Jenkins tells Docker to take the code and package it into a container image using the `docker build` command.
* or i cansay, Jenkins is calling/triggering Docker. Jenkins (via its pipeline) is instructing Docker to build an image using the docker build command.
* Jenkins is the CI/CD orchestrator that runs scripted steps. Docker is the tool being used by Jenkins.
* So, Jenkins is in charge of the automation and Docker is the tool that does the building.
* **Why it's important:** Creating a Docker image helps your app run the same way no matter where it's deployed, which makes testing and deployment more reliable.

### 4. `Jenkins ->> K8s: Deploy YAML`

* **What it means:** Jenkins uses Kubernetes configuration files (YAML files) to deploy the Docker image.
* It runs commands like `kubectl apply -f deployment.yaml` to send the app to the Kubernetes cluster.
* **Why it's important:** This step makes sure the new version of the app is deployed automatically without any manual steps.

### 5. `K8s ->> User: Expose via Service`

* **What it means:** Kubernetes sets up a Service that makes the app accessible to users through the internet or a local network.
* It routes the request from the user's browser to the app running inside a container.
* **Why it's important:** Without this, users wouldn't be able to open the app in a browser or connect to it.

---

## Summary Table

| Actor      | Role                                |
| ---------- | ----------------------------------- |
| Developer  | Writes and pushes code              |
| GitHub     | Triggers Jenkins when code changes  |
| Jenkins    | Automates CI/CD tasks               |
| Docker     | Containerizes the app               |
| Kubernetes | Orchestrates and runs the container |
| User       | Accesses the live application       |

---

> This interpretation helps reinforce the understanding of each component’s role in a modern DevOps CI/CD pipeline.


## 🔗 What’s Next?

* Add Helm charts for reusability
* Configure readiness/liveness probes
* Deploy to AWS/GCP

---

## 🔗 Questions or Feedback?

This project is part of my DevOps learning journey. Fork it, build on it, and feel free to reach out with suggestions!

> Built with motivation by Rubaiya
