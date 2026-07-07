# DevOps CI/CD Pipeline using Jenkins, Docker, and Kubernetes

## 🚀 Project Objective

Design and automate a robust CI/CD pipeline to deploy a multi-tier web application (frontend and backend) using:

- **Jenkins** for Continuous Integration and Continuous Deployment
- **Docker** for containerization
- **Kubernetes** for orchestration (via **Minikube**)
- **AWS EC2** (c7i-flex.large) as the host environment
- **GitHub** as the code repository

The goal is to build a reliable, production-ready DevOps workflow.

---

## 🌐 Hosting Environment

- **Cloud Platform**: AWS EC2
- **Instance Type**: c7i-flex.large (2 vCPU, 4 GB RAM)
- **Operating System**: Ubuntu 24.04 LTS
- **Security Group Ports Opened**: 22 (SSH), 8080 (Jenkins), 30080 (Backend), 30081 (Frontend)
- **Minikube Tunnel**: Enabled using `minikube tunnel --bind-address 0.0.0.0`

---

## 🛠️ Tools & Technologies

| Tool       | Purpose                            |
| ---------- | ---------------------------------- |
| GitHub     | Version Control                    |
| Docker     | Containerize frontend/backend apps |
| Jenkins    | Automate CI/CD pipeline            |
| Kubernetes | Manage containerized workloads     |
| Minikube   | Local Kubernetes cluster           |
| Nginx      | Frontend static file server        |
| Flask      | Python backend web server          |

---

## 📁 Project Structure

```
DevOps-CI-CD-Pipeline-using-Jenkins-Docker-and-Kubernetes/
├── backend/
│   ├── Dockerfile
│   ├── app.py
│   └── requirements.txt
├── frontend/
│   ├── Dockerfile
│   └── index.html
├── k8s/
│   ├── backend-deployment.yaml
│   ├── frontend-deployment.yaml
│   └── service.yaml
├── Jenkinsfile
├── docker-compose.yml
└── README.md
```

---

## ⚙️ Setup and Installation

### 🔧 Install Required Tools

```bash
sudo apt update && sudo apt install -y git curl docker.io
sudo usermod -aG docker $USER
```

Install Minikube:

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

Install kubectl:

```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl && sudo mv kubectl /usr/local/bin/
```

Start Minikube:

```
minikube start --driver=docker
minikube tunnel --bind-address 0.0.0.0
```

---

## 📦 Docker: Containerization

## Backend Dockerfile
```
FROM python:3.9-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install Flask==2.0.3 Werkzeug==2.0.3
COPY app.py .
EXPOSE 5000
CMD ["python", "app.py"]
```
## Frontend Dockerfile
```
FROM nginx:alpine
COPY index.html /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```
### Build & Push Images
```
docker build -t shivakumar46/devops-backend:latest ./backend
docker build -t shivakumar46/devops-frontend:latest ./frontend

docker push shivakumar46/devops-backend:latest
docker push shivakumar46/devops-frontend:latest

```
### Apply Resources

```
kubectl apply -f k8s/
kubectl get pods -o wide
kubectl get svc
```

---

## 🔄 Jenkins CI/CD Pipeline

### Jenkinsfile
---
pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')
    }

    stages {
        stage('Checkout Code from GitHub') {
            steps {
                git branch: 'main', url: 'https://github.com/shivakumarshivas/Devops-Project.git'
            }
        }

        stage('Build Backend Image') {
            steps {
                script {
                    docker.build("shivakumar46/devops-backend:latest", "./backend")
                }
            }
        }

        stage('Build Frontend Image') {
            steps {
                script {
                    docker.build("shivakumar46/devops-frontend:latest", "./frontend")
                }
            }
        }

        stage('Login to DockerHub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-creds') {
                        echo 'Logged in to DockerHub'
                    }
                }
            }
        }

        stage('Push Images') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-creds') {
                        docker.image("shivakumar46/devops-backend:latest").push()
                        docker.image("shivakumar46/devops-frontend:latest").push()
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    export KUBECONFIG=/var/lib/jenkins/.kube/config
                    kubectl apply -f k8s/backend-deployment.yaml
                    kubectl apply -f k8s/frontend-deployment.yaml
                    kubectl apply -f k8s/service.yaml
                '''
            }
        }
    }
}
`
---
## ✅ Output Validation

curl Test

```
curl http://<EC2-PUBLIC-IP>:30080    # Backend
curl http://<EC2-PUBLIC-IP>:30081    # Frontend
```

### Jenkins Pipeline

- All stages: Clone > Build > Push > Deploy should show as green ✅


## 📎 Links

- 🔗 GitHub Repository:[Devops-Project]https://github.com/shivakumarshivas/Devops-Project.git)
- 🔗 LinkedIn Profile:[Shiva Kumar]https://www.linkedin.com/in/shiva-kumar11)

---

## 📚 Conclusion

This project is a full-fledged demonstration of how to containerize applications, automate deployments using Jenkins, and orchestrate containers via Kubernetes — all deployed on an AWS EC2 instance. It proves your ability to integrate multiple DevOps tools in a production-like setup.

