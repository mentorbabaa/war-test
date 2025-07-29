🚀 Java WAR CI/CD Pipeline to EC2 with GitHub Actions

This project demonstrates a production-ready CI/CD pipeline to build, containerize, and deploy a Java WAR-based application to an AWS EC2 instance using GitHub Actions and DockerHub.

📁 Repository Structure

.
├── .github
│   └── workflows
│       ├── build.yml        # CI: Build, Dockerize & Push to DockerHub
│       └── deploy.yml       # CD: Deploy image to EC2 via SSH
├── Dockerfile               # Docker build for WAR file
├── pom.xml                  # Maven project descriptor
└── src/                     # Java source code

⚙️ CI/CD Overview

🔨 CI — build.yml

Trigger: On push to master

Actions:

Checkout repo

Build WAR using Maven

Dockerize the WAR file

Push Docker image to DockerHub

🚀 CD — deploy.yml

Trigger: On successful completion of build.yml

Actions:

SSH into EC2

Pull latest Docker image

Stop/remove old container (if exists)

Run new container on port 8080

🔑 Required GitHub Secrets

Go to your repo → Settings → Secrets and variables → Actions → Repository secrets and add:

Secret Name

Description

DOCKERHUB_USERNAME

Your DockerHub username

DOCKERHUB_TOKEN

DockerHub access token

EC2_HOST

Public IP or DNS of the EC2 instance

EC2_USER

SSH username (e.g., ubuntu, ec2-user)

EC2_SSH_KEY

Private key for SSH (use multi-line secret)

🐳 Dockerfile (Sample)

FROM tomcat:9.0-jdk17-temurin
COPY target/*.war /usr/local/tomcat/webapps/ROOT.war

Customize the Dockerfile based on your WAR structure if needed.

🚀 EC2 Setup Instructions

Launch an EC2 instance (Amazon Linux 2 / Ubuntu preferred)

Install Docker:

sudo yum update -y       # or sudo apt update -y
sudo yum install docker  # or sudo apt install docker.io
sudo service docker start
sudo usermod -aG docker $USER

Open port 8080 in your EC2 Security Group

Add your .pem private key to GitHub Secrets as EC2_SSH_KEY

Add EC2 public IP as EC2_HOST secret

🧪 Testing the Deployment

Push code to master branch:

git add .
git commit -m "Trigger CI/CD"
git push origin master

GitHub Actions will:

Build & push Docker image

SSH into EC2 and deploy

Access the app:

http://<EC2_PUBLIC_IP>:8080

✅ Sample GitHub Actions Workflows

.github/workflows/build.yml

name: Build and Push

on:
  push:
    branches: [ "master" ]
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Set up Java
        uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '17'

      - name: Build with Maven
        run: mvn clean package

      - name: Build Docker Image
        run: |
          docker build -t ${{ secrets.DOCKERHUB_USERNAME }}/war-test-app:${{ github.sha }} .

      - name: Login to DockerHub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Push Docker Image
        run: |
          docker push ${{ secrets.DOCKERHUB_USERNAME }}/war-test-app:${{ github.sha }}

.github/workflows/deploy.yml

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
    - name: Deploy to EC2
      uses: appleboy/ssh-action@v1.0.0
      with:
        host: ${{ secrets.EC2_HOST }}
        username: ${{ secrets.EC2_USER }}
        key: ${{ secrets.EC2_SSH_KEY }}
        script: |
          docker pull ${{ secrets.DOCKERHUB_USERNAME }}/war-test-app:${{ github.sha }}
          docker stop war-test || true
          docker rm war-test || true
          docker run -d --name war-test -p 8080:8080 --restart unless-stopped ${{ secrets.DOCKERHUB_USERNAME }}/war-test-app:${{ github.sha }}

📌 Tips

Make sure the DockerHub repo is public or your EC2 instance can docker login before pulling.

Add a health check endpoint to your app for better post-deployment verification.

Use separate EC2s for staging and production.

🧰 Tools Used

Java 17 / Maven

Docker

GitHub Actions

DockerHub

AWS EC2

SSH via Appleboy GitHub Action
