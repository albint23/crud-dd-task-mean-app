# 📌 Discover Dollar – DevOps Assignment

**MEAN Application | Docker | GitHub Actions CI/CD | Nginx Reverse Proxy | Cloud Deployment**

This repository contains the completed assignment for the **DevOps Engineer Intern** role at Discover Dollar.  
The objective was to containerize, deploy, automate, and document a full-stack MEAN application using Docker, GitHub Actions, Nginx, and an Ubuntu cloud VM.

---

# 🏗️ Architecture Overview

```
GitHub → GitHub Actions → Docker Hub → AWS EC2 → Docker Compose → Nginx → MEAN App
```

Components included:

- Angular Frontend (Dockerized + served via Nginx)  
- Node.js + Express Backend (Dockerized)  
- MongoDB (Docker-based)  
- Docker Compose (multi-service)  
- GitHub Actions CI/CD Pipeline  
- Nginx Reverse Proxy (inside container)  
- Ubuntu VM (AWS EC2 – 51.20.55.48)

---

# 🐳 1. Dockerfiles

Below are the exact Dockerfiles used in this project.

---

## 🔹 Frontend Dockerfile (`frontend/Dockerfile`)

```dockerfile
FROM node:18-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build --prod

# --- Stage 2: Nginx ---
FROM nginx:alpine
COPY nginx.conf /etc/nginx/conf.d/default.conf
COPY --from=build /app/dist/angular-15-crud /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

---

## 🔹 Backend Dockerfile (`backend/Dockerfile`)

```dockerfile
FROM node:18-alpine
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 8080
CMD ["node", "server.js"]
```

---

# 🧩 2. Docker Compose Setup

```yaml
version: "3.9"

services:
  mongo:
    image: mongo:6
    container_name: mongo
    ports:
      - "27017:27017"
    volumes:
      - mongo_data:/data/db

  backend:
    image: albin666/mean-backend:latest
    container_name: backend
    environment:
      - MONGO_URL=mongodb://mongo:27017/dd_db
    ports:
      - "8080:8080"
    depends_on:
      - mongo

  frontend:
    image: albin666/mean-frontend:latest
    container_name: frontend
    ports:
      - "80:80"
    depends_on:
      - backend

volumes:
  mongo_data:
```

---

# 🌐 3. Nginx Reverse Proxy

**File:** `frontend/nginx.conf`

```nginx
server {
    listen 80;

    # Proxy API calls to backend
    location /api/ {
        proxy_pass http://backend:8080/;
    }

    # Serve Angular app
    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

### View Nginx Config (inside container)
```
docker exec -it frontend cat /etc/nginx/conf.d/default.conf
```

### Check Nginx Status
```
docker exec -it frontend ps aux
```

---

# 🤖 4. CI/CD – GitHub Actions

**File:** `.github/workflows/deploy.yml`

```yaml
name: CI/CD - Build, Push, Deploy

on:
  push:
    branches: ["main"]

jobs:
  build-and-push:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build & Push Backend
        run: |
          docker build -t albin666/mean-backend:latest backend
          docker push albin666/mean-backend:latest

      - name: Build & Push Frontend
        run: |
          docker build -t albin666/mean-frontend:latest frontend
          docker push albin666/mean-frontend:latest

  deploy:
    runs-on: ubuntu-latest
    needs: build-and-push

    steps:
      - name: Deploy to EC2
        uses: appleboy/ssh-action@v1.2.0
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ubuntu
          key: ${{ secrets.EC2_SSH_KEY }}
          script: |
            cd ~/project
            git pull
            docker compose pull
            docker compose down
            docker compose up -d
```

---

# 🌍 5. Application Access Details

| Service | URL |
|--------|------|
| **Frontend Application** | http://51.20.55.48 |
---

# 📸 6. Required Screenshots  
(All stored in the `/screenshots` folder)

|

---

# 🧠 7. Issues & Fixes

- Fixed Docker permission issue using `sudo usermod -aG docker $USER`  
- Angular build path adjusted to `dist/angular-15-crud`  
- Nginx routing corrected for `/api/`  
- GitHub Actions SSH deploy corrected  
- Docker networking fixed for MongoDB  

---

# 🎯 8. Final Result

The MEAN application is now:

- Fully containerized  
- Deployed on AWS EC2 using Docker Compose  
- Publicly accessible at **http://51.20.55.48**  
- Auto-updated through GitHub Actions  
- Reverse-proxied through Nginx (inside container)  
- Fully functional with backend API + MongoDB  

This completes the **Discover Dollar DevOps Assignment**.

