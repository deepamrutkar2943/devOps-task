# MEAN Tutorials CRUD – DevOps Task

Full-stack **MEAN** application that manages tutorials (title, description, published).  
This repo contains:

- **Angular 15** frontend  
- **Node.js + Express** backend  
- **MongoDB** database  
- **Dockerized** services (frontend, backend, Mongo, Nginx)  
- **GitHub Actions** CI pipeline to build and push Docker images  
- **AWS EC2** setup for deployment with Docker Compose + Nginx reverse proxy

---

## 1. Project Structure

```text
.
├── backend/           # Node.js + Express REST API
├── frontend/          # Angular 15 client app
├── nginx/
│   └── default.conf   # Nginx reverse proxy config
├── docker-compose.yml # Multi-container stack (Mongo, backend, frontend, Nginx)
└── .github/workflows/
    └── ci-cd.yml      # GitHub Actions pipeline
```
## 2. Core Features

Create / Read / Update / Delete tutorials
Frontend ↔ Backend communication via REST API
Single entry point for users via Nginx on port 80

## 3. Local Development (without Docker)
### 3.1 Backend (Express + MongoDB)
```
cd backend
npm install
# db config: app/config/db.config.js
# default: mongodb://127.0.0.1:27017/tutorial_db
node server.js
```

Backend runs on:
``` http://localhost:8080/api/tutorials ```

#### 3.2 Frontend (Angular 15)
```
cd frontend
npm install
ng serve --port 8081
```

Frontend runs on:
``` http://localhost:8081/ ```

## 4. Docker & Nginx Architecture

Docker Compose runs 4 services:

- mongo – MongoDB database

- backend – Node.js API (talks to mongo)

- frontend – Angular app (served via ng serve)

- nginx – Reverse proxy that exposes:

  / → Angular frontend

  /api/ → Express backend

### 4.1 Key Files
backend/app/config/db.config.js
```
module.exports = {
  url: process.env.MONGO_URI || "mongodb://127.0.0.1:27017/tutorial_db",
};
```
## 5. Running the Stack with Docker Compose

These commands are meant for machines where Docker is available (e.g. EC2).
```
docker-compose up -d --build
docker ps
```
App will be available at:
```http://<VM_IP>/```

## 6. AWS EC2 Deployment (Manual Steps)

High-level steps I followed to deploy on AWS Free Tier EC2:

Create EC2 Instance

OS: Ubuntu 22.04

Type: t2.micro (free tier)

Open ports: 22 (SSH), 80 (HTTP)

Install Docker & Docker Compose
```
sudo apt update
sudo apt install -y docker.io
# docker-compose via binary
sudo apt install -y curl
sudo curl -L "https://github.com/docker/compose/releases/download/v2.24.5/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker --version
docker-compose --version
```

Clone Repo on EC2
```
cd ~
git clone https://github.com/<your-username>/devOps-task.git
cd devOps-task
```
  
Deploy Latest Images
```
cd ~/devOps-task
docker-compose pull
docker-compose up -d
docker ps
```
Access app in browser:
``` http://<EC2_PUBLIC_IP>/```

## 7. CI Pipeline (GitHub Actions)

Location: .github/workflows/ci-cd.yml

On every push to main (or manual trigger):

Checkout repo

Login to Docker Hub using repo secrets:

DOCKERHUB_USERNAME

DOCKERHUB_TOKEN

Build & push backend image
→ ${DOCKERHUB_USERNAME}/mean-tutorials-backend:latest

Build & push frontend image
→ ${DOCKERHUB_USERNAME}/mean-tutorials-frontend:latest

SSH deploy step to EC2 (see note below)

Secrets used

DOCKERHUB_USERNAME – Docker Hub username

DOCKERHUB_TOKEN – Docker Hub access token with read/write

SSH_HOST – EC2 public IP (for SSH deploy, WIP)

SSH_USER – ubuntu

SSH_PRIVATE_KEY – contents of the EC2 .pem key

## 8. Known Issue :

The GitHub Actions SSH deploy step to EC2 is partially configured and currently flaky due to SSH connectivity/permissions on the t2.micro instance. For now, I am using:
```
git pull origin main
docker-compose pull
docker-compose up -d --remove-orphans

```
directly on EC2 to deploy the latest images. In the next iteration, I plan to stabilize the SSH action so that the pipeline can fully auto-deploy to EC2 without any manual commands.
