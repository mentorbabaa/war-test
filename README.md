# 🚀 Java WAR CI/CD Pipeline to AWS EC2 using GitHub Actions

This repository demonstrates a production-ready CI/CD pipeline to build, containerize, and deploy a Java WAR application to an AWS EC2 instance using GitHub Actions and DockerHub.

---

📂 Repository Structure


<img width="800" height="300" alt="image" src="https://github.com/user-attachments/assets/e28a47fd-5e15-4479-93f4-c2699cf542aa" />


---

## 🔄 CI/CD Pipeline Overview

- **build.yml**  
  Trigger: On push to `master` branch  
  Actions:
  - Checkout repository
  - Build WAR using Maven
  - Build Docker image tagged with Git SHA
  - Push Docker image to DockerHub

- **deploy.yml**  
  Trigger: After successful completion of `build.yml`  
  Actions:
  - SSH into EC2 instance
  - Pull the latest Docker image
  - Stop and remove existing container (if any)
  - Run new Docker container on port 8080

---

## 🔑 Required GitHub Secrets

Go to your GitHub repo → **Settings** → **Secrets and variables** → **Actions** → **Repository secrets** and add the following:

| Secret Name           | Description                                  |
| --------------------- | -------------------------------------------- |
| `DOCKERHUB_USERNAME`  | Your DockerHub username                       |
| `DOCKERHUB_TOKEN`     | DockerHub Access Token (create from DockerHub settings) |
| `EC2_HOST`            | Public IP address or DNS of your EC2 instance |
| `EC2_USER`            | SSH username for EC2 instance (e.g., `ubuntu`, `ec2-user`) |
| `EC2_SSH_KEY`         | Private SSH key corresponding to EC2 (paste full key contents) |

---

## 🐳 Dockerfile Example

```dockerfile
FROM tomcat:9.0-jdk17-temurin
COPY target/*.war /usr/local/tomcat/webapps/ROOT.war
```

## 🖥️ EC2 Setup Instructions
1. Launch an EC2 instance (Amazon Linux 2 or Ubuntu recommended).

Install Docker on the EC2 instance:
- sudo yum update -y        # or sudo apt update -y
- sudo yum install docker   # or sudo apt install docker.io
- sudo service docker start
- sudo usermod -aG docker $USER

2. Open port 8080 in the EC2 Security Group inbound rules.

3. Add your private SSH key (.pem) to GitHub Secrets as EC2_SSH_KEY.

4. Set EC2 public IP or DNS as EC2_HOST.

5. Set SSH user as EC2_USER (ubuntu or ec2-user depending on AMI).

## 🚀 How to Trigger Deployment
Push code changes to the master branch:

```
git add .
git commit -m "Trigger CI/CD pipeline"
git push origin master
```
This will:

Trigger the build pipeline to build & push Docker image.

Automatically trigger deployment workflow that deploys the new container to your EC2.

## 🔄 GitHub Actions Workflow Samples
build.yml
```
name: Build and Push

on:
  push:
    branches: [ "master" ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '17'

      - run: mvn clean package

      - run: docker build -t ${{ secrets.DOCKERHUB_USERNAME }}/war-test-app:${{ github.sha }} .

      - uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - run: docker push ${{ secrets.DOCKERHUB_USERNAME }}/war-test-app:${{ github.sha }}
```

deploy.yml
```
name: Deploy to EC2

on:
  workflow_run:
    workflows: ["Build and Push"]
    types:
      - completed

jobs:
  deploy:
    if: ${{ github.event.workflow_run.conclusion == 'success' }}
    runs-on: ubuntu-latest

    steps:
      - uses: appleboy/ssh-action@v1.0.0
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.EC2_SSH_KEY }}
          script: |
            docker pull ${{ secrets.DOCKERHUB_USERNAME }}/war-test-app:${{ github.sha }}
            docker stop war-test || true
            docker rm war-test || true
            docker run -d --name war-test -p 8080:8080 --restart unless-stopped ${{ secrets.DOCKERHUB_USERNAME }}/war-test-app:${{ github.sha }}
```

## 🧰 Tools Used

Java 17 / Maven
Docker
GitHub Actions
DockerHub
AWS EC2
SSH via Appleboy GitHub Action

Enjoy !!! Powered by Mentorbabaa
