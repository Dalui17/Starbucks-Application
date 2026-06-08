# 🚀 Enterprise CI/CD Pipeline using Jenkins, Docker & AWS EC2
## Complete Setup Guide

---

# Table of Contents

1. Project Overview
2. Architecture
3. Prerequisites
4. AWS EC2 Setup
5. Install Java
6. Install Jenkins
7. Initial Jenkins Configuration
8. Install Git
9. Install Docker
10. Configure Docker Permissions
11. Install Jenkins Plugins
12. Configure NodeJS
13. Clone Application Repository
14. Configure Docker Hub Credentials
15. Create CI Pipeline
16. CI Pipeline Explanation
17. Create CD Pipeline
18. CD Pipeline Explanation
19. Deployment Verification
20. Screenshots Section
21. Troubleshooting
22. Interview Questions
23. Learning Outcomes
24. Conclusion

---

# 1. Project Overview

This project demonstrates a complete CI/CD implementation using:

- GitHub
- Jenkins
- Docker
- Docker Hub
- AWS EC2

The goal is to automate the complete software delivery lifecycle from source code commit to deployment.

---

# 2. Architecture


```text
## Enterprise Architecture Flow

```text
Developer
    │
    ▼
GitHub Repository
    │
    ▼
Jenkins CI Pipeline
    │
    ├── Checkout Source Code
    ├── Install Dependencies
    ├── Build Application
    ├── Build Docker Image
    └── Push Docker Image
    │
    ▼
Docker Hub Registry
    │
    ▼
Jenkins CD Pipeline
    │
 ┌──┴────────────┐
 ▼               ▼
STAGING        PROD
Deployment   Deployment
    │             │
    └──────┬──────┘
           ▼
      AWS EC2
```

## Flow

Developer
↓
GitHub Repository
↓
Jenkins CI Pipeline
↓
Docker Hub
↓
Jenkins CD Pipeline
↓
AWS EC2
↓
Live Application

---

# 3. Prerequisites

Required Accounts:

- AWS Account
- GitHub Account
- Docker Hub Account

Required Knowledge:

- Basic Linux Commands
- Basic Git Commands
- Basic Docker Concepts

---

# 4. AWS EC2 Setup

Launch Instance

Name:
jenkins-server

AMI:
Ubuntu Server 24.04 LTS

Instance Type:
t2.medium

Storage:
30 GB

Security Group:

Port 22 - SSH

Port 8080 - Jenkins

Port 3000 - Application

Connect:

```bash
ssh -i your-key.pem ubuntu@PUBLIC-IP
```

---

# 5. Install Java

```bash
sudo apt update

sudo apt install fontconfig openjdk-21-jre -y

java -version
```

Verify Java installation before continuing.

---

# 6. Install Jenkins

Add Jenkins Repository Key

```bash
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key
```

Add Repository

```bash
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
```

Install Jenkins

```bash
sudo apt update

sudo apt install jenkins -y
```

Start Jenkins

```bash
sudo systemctl enable jenkins

sudo systemctl start jenkins
```

Check Status

```bash
sudo systemctl status jenkins
```

---

# 7. Initial Jenkins Configuration

Retrieve Password

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Open:

http://PUBLIC-IP:8080

Choose:

Install Suggested Plugins

Create Admin User

Login to Jenkins Dashboard

---

# 8. Install Git

```bash
sudo apt install git -y

git --version
```

---

# 9. Install Docker

```bash
sudo apt install docker.io -y

sudo systemctl enable docker

sudo systemctl start docker

docker --version
```

Verify

```bash
sudo systemctl status docker
```

---

# 10. Configure Docker Permissions

```bash
sudo usermod -aG docker jenkins

sudo usermod -aG docker ubuntu
```

Restart Jenkins

```bash
sudo systemctl restart jenkins
```

Verify

```bash
groups jenkins
```

Docker group should be visible.

---

# 11. Install Jenkins Plugins

Navigate:

Manage Jenkins
→ Plugins

Install:

- Eclipse Temurin Installer
- NodeJS
- Docker
- Docker Commons
- Docker Pipeline
- Docker API
- Docker Build Step
- Pipeline Stage View
- Git
- Pipeline

Restart Jenkins after installation.

---

# 11.1 Configure Docker Hub Credentials

Navigate:

Manage Jenkins
→ Credentials
→ System
→ Global Credentials
→ Add Credentials

Configure:

```text
Kind        : Username with Password
Username    : DockerHub Username
Password    : DockerHub Password / Access Token
ID          : docker
Description : docker
```

This credential will be used by Jenkins pipelines to authenticate with Docker Hub and push images securely.

---

# 11.2 Jenkins Global Tool Configuration

Navigate:

Manage Jenkins
→ Tools

Configure the following tools.

## JDK Configuration

```text
Name : jdk17
```

Enable:

```text
Install Automatically
```

Installer:

```text
Install from adoptium.net
```

Version:

```text
jdk-17.0.8.1+1
```

---

## NodeJS Configuration

```text
Name : node16
```

Enable:

```text
Install Automatically
```

Version:

```text
NodeJS 16.20.0
```

Used during:

```bash
npm install
npm run build
```

---

## Docker Configuration

```text
Name : docker
```

Enable:

```text
Install Automatically
```

Version:

```text
latest
```

Used during:

```bash
docker build
docker tag
docker push
docker pull
docker run
```

---

# 12. Configure NodeJS

Navigate:

Manage Jenkins
→ Tools
→ NodeJS Installations

Configure:

Name: node16

Version: NodeJS 16

Save.

---

# 13. Clone Application Repository

```bash
git clone https://github.com/Dalui17/Starbucks-Application.git

cd Starbucks-Application
```

Verify Files

```bash
ls
```

---

# 14. Configure Docker Hub Credentials

Navigate:

Manage Jenkins
→ Credentials
→ Global Credentials
→ Add Credentials

Type:

Username with Password

Example:

Username: your-dockerhub-user

Password: your-dockerhub-password

ID: docker

Save.

---

# 15. Create CI Pipeline

Create New Item

Name:

starbucks-app-ci

Type:

Pipeline

Configuration:

Pipeline script from SCM

SCM:

Git

Repository:

https://github.com/Dalui17/Starbucks-Application.git

Branch:

*/main

Save.

---

# 16. CI Pipeline Explanation

Pipeline Stages

1. Declarative Checkout SCM
2. Tool Install
3. Clean Workspace
4. Git Checkout
5. Install NPM Dependencies
6. Build Docker Image
7. Tag Docker Image
8. Push Docker Image to Docker Hub

Benefits:

- Automated Build
- Consistent Packaging
- Artifact Versioning
- Faster Delivery

Expected Result:

SUCCESS

Docker image available in Docker Hub.

---

# 17. Create CD Pipeline

Create New Item

Name:

starbucks-app-cd

Type:

Pipeline

Enable:

Build after other projects are built

Upstream Project:

starbucks-app-ci

Save.

---

# 17.1 Configure Parameterized CD Pipeline

Enable:

```text
This project is parameterized
```

Add the following Boolean Parameters.

### STAGING

```text
Name        : STAGING
Description : staging ip
```

### PROD

```text
Name        : PROD
Description : prod ip
```

Purpose:

These parameters allow deployment to multiple environments using the same CD pipeline.

Environment Options:

```text
STAGING
PROD
```

This is commonly used in enterprise CI/CD implementations where deployment targets differ between testing and production environments.

---

# 18. CD Pipeline Explanation

Pipeline Stages

1. Pull Latest Image
2. Stop Existing Container
3. Remove Existing Container
4. Deploy New Container
5. Verify Application

Deployment Commands

```bash
docker stop starbucks || true

docker rm starbucks || true

docker pull YOUR_DOCKERHUB_USERNAME/starbucks:latest

docker run -d --name starbucks -p 3000:3000 YOUR_DOCKERHUB_USERNAME/starbucks:latest
```

Benefits

- Automated Deployment
- Zero Manual Steps
- Faster Releases
- Reduced Errors

---

# 18.1 Configure Downstream Trigger

Navigate:

CD Pipeline
→ Configure
→ Triggers

Enable:

```text
Build after other projects are built
```

Projects to watch:

```text
CI pipeline
```

Select:

```text
Trigger only if build is stable
```

Purpose:

Automatically start the CD Pipeline after successful completion of the CI Pipeline.

This removes manual intervention and creates a fully automated deployment workflow.

---

# 18.2 CI → CD Relationship

Workflow:

```text
GitHub Push
        │
        ▼
CI Pipeline
        │
        ▼
Checkout Source Code
        │
        ▼
Install Dependencies
        │
        ▼
Build Docker Image
        │
        ▼
Push Docker Image
        │
        ▼
Docker Hub
        │
        ▼
Trigger CD Pipeline
        │
   ┌────┴────┐
   ▼         ▼
STAGING    PROD
        │
        ▼
Application Deployment
```

Benefits:

- Complete Automation
- Faster Releases
- Reduced Human Error
- Consistent Deployments
- Enterprise Deployment Workflow

---

# 19. Deployment Verification

Check Running Containers

```bash
docker ps
```

Expected:

Container running on Port 3000

Open Browser

http://PUBLIC-IP:3000

Application should load successfully.

---

# 20. Screenshots Section

Add screenshots in:

screenshots/

Recommended Files:

banner.png

architecture.png

jenkins-dashboard.png

ci-pipeline-success.png

dockerhub-push.png

cd-pipeline-success.png

deployed-application.png

Example:

![CI Pipeline](../screenshots/ci-pipeline-success.png)

---

# 21. Troubleshooting

## Docker Permission Denied

```bash
sudo usermod -aG docker jenkins

sudo systemctl restart jenkins
```

## Jenkins Not Opening

```bash
sudo systemctl status jenkins

sudo systemctl restart jenkins
```

## Docker Push Failed

Verify:

- DockerHub Credentials
- Username
- Password / Token

## Port 3000 Not Accessible

Verify:

```bash
docker ps
```

Check EC2 Security Group.

## Git Clone Failed

Verify:

- Internet Connectivity
- Git Installation
- Repository URL

---

# 22. Interview Questions

## What is CI?

Continuous Integration is the process of automatically integrating and validating code changes.

## What is CD?

Continuous Delivery automates deployment processes.

## Why Jenkins?

Jenkins automates build, test, and deployment tasks.

## Why Docker?

Docker packages applications into portable containers.

## What is Docker Hub?

Docker Hub is a container image registry.

## What is a Jenkins Pipeline?

A Jenkins Pipeline defines software delivery stages as code.

## Why Separate CI and CD?

Provides better security and deployment control.

## What is a Docker Image?

A read-only template used to create containers.

## What is a Container?

A running instance of a Docker image.

## What happens after git push?

Jenkins triggers CI, builds image, pushes image, triggers deployment.

---

# Advanced Deployment Scenarios

## Scenario 1: Docker Hub Repository is Private

If the Docker Hub repository is private, Jenkins must authenticate before pulling images.

Steps:

1. Store Docker Hub credentials in Jenkins.
2. Perform Docker login.
3. Pull image from private repository.
4. Deploy container.

---

## Scenario 2: Deploying to Remote Server

Instead of deploying on the Jenkins server itself, Jenkins can connect to a remote EC2 instance using SSH.

Flow:

Jenkins → SSH → Remote EC2 → Docker Pull → Docker Run

Benefits:

- Better separation of concerns
- More secure architecture
- Production-ready deployment model

---

## Scenario 3: Private Repository + Remote Deployment

Jenkins authenticates with Docker Hub and deploys containers on a remote EC2 server through SSH.
---

# 23. Learning Outcomes

After completing this project you will understand:

- Jenkins Administration
- Jenkins Pipeline Creation
- Docker Installation
- Docker Image Lifecycle
- Docker Hub Registry
- AWS EC2 Administration
- Continuous Integration
- Continuous Delivery
- Deployment Automation
- Credential Management
- DevOps Best Practices

---

# 24. Conclusion

In this project we successfully implemented an end-to-end CI/CD pipeline using Jenkins, Docker, GitHub, Docker Hub, and AWS EC2.

The solution automates application build, packaging, image storage, and deployment, reducing manual effort while improving consistency and reliability.

This project serves as a strong foundation for production-grade DevOps workflows and demonstrates practical CI/CD implementation.

---

# Author

Anirban Dalui

DevOps Engineer | AWS | Docker | Jenkins | Kubernetes | Terraform | Ansible
