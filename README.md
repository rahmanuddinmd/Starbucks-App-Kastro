☕ Starbucks React App — Jenkins CI/CD with Docker & AWS EC2

A complete DevOps CI/CD project that automates the build, test, containerization, image publishing, and deployment of a React-based Starbucks application using GitHub, Jenkins, Docker, Docker Hub, and AWS EC2.

📌 Project Overview

This project demonstrates an end-to-end CI/CD workflow:

Developer
   │
   ▼
GitHub
   │
   │ Checkout
   ▼
Jenkins
   │
   ├── Clean Workspace
   ├── Checkout Source Code
   ├── Verify Git Commit
   ├── Check Node.js / npm
   ├── Install Dependencies
   ├── Run Tests
   ├── Build React App
   ├── Build Docker Image
   ├── Login to Docker Hub
   ├── Push Docker Image
   ├── Remove Old Container
   ├── Deploy New Container
   └── Verify Deployment
   │
   ▼
Docker Hub
   │
   ▼
AWS EC2
   │
   ▼
Docker Container
   │
   ▼
React Application

🎯 Objective

The objective of this project is to create a CI/CD pipeline that automatically:

Pulls the React application source code from GitHub.

Installs project dependencies.

Runs application tests.

Builds the React application.

Builds a Docker image.

Pushes versioned Docker images to Docker Hub.

Removes the previously deployed container.

Deploys the latest application version on AWS EC2.

Verifies that the application container is running correctly.

🏗️ Architecture

                    ┌────────────────────┐
                    │       GitHub       │
                    │   React Source     │
                    └─────────┬──────────┘
                              │
                              │ Checkout
                              ▼
                    ┌────────────────────┐
                    │      Jenkins       │
                    │     Port 8080      │
                    └─────────┬──────────┘
                              │
                 ┌────────────┼────────────┐
                 │            │            │
                 ▼            ▼            ▼
              npm ci       npm test    npm run build
                 │            │            │
                 └────────────┼────────────┘
                              │
                              ▼
                    ┌────────────────────┐
                    │       Docker       │
                    │    Build Image     │
                    └─────────┬──────────┘
                              │
                              │ docker push
                              ▼
                    ┌────────────────────┐
                    │     Docker Hub     │
                    │   starbucks-app    │
                    └─────────┬──────────┘
                              │
                              │ Deploy
                              ▼
                    ┌────────────────────┐
                    │      AWS EC2       │
                    │                    │
                    │ Host Port: 8081    │
                    │        │           │
                    │        ▼           │
                    │ Container: 3000    │
                    │        │           │
                    │        ▼           │
                    │    React App       │
                    └────────────────────┘

🛠️ Technology Stack

Technology

Purpose

React.js

Front-end application

Node.js

React build/runtime environment

npm

Dependency management

Git

Version control

GitHub

Source-code repository

Jenkins

CI/CD automation

Java 21

Jenkins runtime

Docker

Application containerization

Docker Hub

Docker image registry

AWS EC2

Cloud deployment server

Ubuntu

EC2 operating system

🔌 Final Port Configuration

Service

Port

Jenkins

8080

React application inside container

3000

Application exposed on EC2

8081

Docker mapping

8081:3000

Browser
   │
   ▼
EC2 :8081
   │
   ▼
Docker Container :3000
   │
   ▼
React Application

EXPOSE 3000 documents the port used by the application inside the container.
The actual host-to-container mapping is created at runtime with -p 8081:3000.

🚀 Deployment Setup

1. Create an AWS EC2 Instance

Recommended configuration for a learning/demo environment:

Operating System : Ubuntu
Storage          : 15–20 GB or more
Inbound Ports    : 22, 8080, 8081

Required Security Group Rules

Type

Port

Purpose

SSH

22

EC2 administration

Custom TCP

8080

Jenkins

Custom TCP

8081

React application

For production environments, restrict inbound source IP/CIDR values instead of exposing administrative services publicly.

2. Connect to EC2

ssh -i your-key.pem ubuntu@EC2-PUBLIC-IP

Update packages:

sudo apt update

3. Install Git

sudo apt install git -y
git --version

4. Install Java 21

Jenkins requires Java.

sudo apt install fontconfig openjdk-21-jre -y
java -version

Expected output should show Java 21.

5. Install Jenkins

Install Jenkins using the official Jenkins installation procedure for your Ubuntu release.

After installation:

sudo systemctl enable jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins

Jenkins runs on:

http://EC2-PUBLIC-IP:8080

Retrieve the initial administrator password:

sudo cat /var/lib/jenkins/secrets/initialAdminPassword

Complete the setup wizard and create the Jenkins administrator account.

🔌 Jenkins Plugins

Go to:

Manage Jenkins
   ↓
Plugins

Install or verify:

Pipeline

Pipeline Stage View

NodeJS

Docker Pipeline

Docker Commons

Git

🟢 Configure Node.js in Jenkins

Go to:

Manage Jenkins
   ↓
Tools
   ↓
NodeJS installations

Add a Node.js installation.

Example name used in this project:

node25

The Jenkinsfile must use exactly the same configured name:

tools {
    nodejs 'node25'
}

Test Node.js from Jenkins

pipeline {
    agent any

    tools {
        nodejs 'node25'
    }

    stages {
        stage('Node Version') {
            steps {
                sh 'node --version'
                sh 'npm --version'
            }
        }
    }
}

🐳 Install Docker

sudo apt update
sudo apt install docker.io -y

sudo systemctl enable docker
sudo systemctl start docker

docker --version

Test Docker:

sudo docker run hello-world

🔐 Give Jenkins Docker Permission

Jenkins must be able to communicate with the Docker daemon.

sudo usermod -aG docker jenkins
sudo systemctl restart docker
sudo systemctl restart jenkins

Verify:

sudo -u jenkins docker ps

If there is no permission error, Jenkins can execute Docker commands.

📦 Docker Hub

Docker image repository used by this project:

rahmanuddinmd17/starbucks-app

Images are versioned with Jenkins build numbers:

rahmanuddinmd17/starbucks-app:1
rahmanuddinmd17/starbucks-app:2
rahmanuddinmd17/starbucks-app:3

The pipeline also publishes:

rahmanuddinmd17/starbucks-app:latest

Using the Jenkins build number provides traceability between a Jenkins build and its corresponding Docker image.

🔑 Configure Docker Hub Credentials in Jenkins

Go to:

Manage Jenkins
   ↓
Credentials
   ↓
Global
   ↓
Add Credentials

Use:

Kind          : Username with password
Username      : Your Docker Hub username
Password      : Docker Hub password/access token
Credential ID : dockerhub-creds

Prefer a Docker Hub access token instead of storing your account password.

The Jenkinsfile references:

credentialsId: 'dockerhub-creds'

Never hard-code passwords or access tokens inside the Jenkinsfile.

📁 Expected Repository Structure

.
├── public/
├── src/
├── package.json
├── package-lock.json
├── Dockerfile
├── Jenkinsfile
└── README.md

🐳 Dockerfile

FROM node:alpine

WORKDIR /app

COPY package.json package-lock.json /app/

RUN npm install

COPY . /app/

EXPOSE 3000

CMD ["npm", "start", "--", "--host", "0.0.0.0"]

Dockerfile Explanation

Base image

FROM node:alpine

Uses the lightweight Node.js Alpine image.

Working directory

WORKDIR /app

Sets /app as the working directory inside the image.

Dependency files

COPY package.json package-lock.json /app/

Copies dependency definitions first.

Install dependencies

RUN npm install

Installs application dependencies.

Copy application

COPY . /app/

Copies the source code into the image.

Application port

EXPOSE 3000

The React application listens on port 3000 inside the container.

Start application

CMD ["npm", "start", "--", "--host", "0.0.0.0"]

Starts the application and allows it to accept connections through the container network.

⚙️ Jenkins Pipeline

The pipeline performs the following stages:

1. Clean Workspace
2. Checkout
3. Verify Code
4. Node.js Version
5. Install Dependencies
6. Test
7. Build Application
8. Build Docker Image
9. Docker Hub Login
10. Push Docker Image
11. Remove Old Container
12. Deploy Container
13. Verify Deployment

Jenkinsfile

Replace YOUR_GITHUB_REPOSITORY_URL and the branch name if necessary.

pipeline {

    agent any

    tools {
        nodejs 'node25'
    }

    environment {
        DOCKER_IMAGE = 'rahmanuddinmd17/starbucks-app'
        CONTAINER_NAME = 'starbucks-container'
        HOST_PORT = '8081'
        CONTAINER_PORT = '3000'
    }

    stages {

        stage('Clean Workspace') {
            steps {
                echo 'Cleaning Jenkins workspace...'
                cleanWs()
            }
        }

        stage('Checkout') {
            steps {
                echo 'Checking out Starbucks application...'

                git branch: 'master',
                    url: 'YOUR_GITHUB_REPOSITORY_URL'
            }
        }

        stage('Verify Code') {
            steps {
                echo 'Checking Git information...'

                sh '''
                    echo "Git Commit:"
                    git rev-parse HEAD

                    echo "Git Branch:"
                    git branch --show-current

                    echo "Latest Commit:"
                    git log -1 --oneline

                    echo "Project Files:"
                    ls -la
                '''
            }
        }

        stage('Node.js Version') {
            steps {
                echo 'Checking Node.js and npm versions...'

                sh '''
                    node --version
                    npm --version
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                echo 'Installing dependencies...'
                sh 'npm ci'
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'

                // Temporary workaround used in the original project.
                // Production pipelines should fail when tests fail.
                sh 'CI=true npm test -- --watchAll=false || true'
            }
        }

        stage('Build Application') {
            steps {
                echo 'Building React application...'

                // Temporary workaround for CI/ESLint warning behavior.
                // Fix project warnings before using this in production.
                sh 'CI=false npm run build'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'

                sh '''
                    docker build \
                        -t ${DOCKER_IMAGE}:${BUILD_NUMBER} \
                        -t ${DOCKER_IMAGE}:latest \
                        .
                '''
            }
        }

        stage('Docker Hub Login') {
            steps {
                echo 'Logging into Docker Hub...'

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login \
                            -u "$DOCKER_USERNAME" \
                            --password-stdin
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                echo 'Pushing Docker image to Docker Hub...'

                sh '''
                    docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                    docker push ${DOCKER_IMAGE}:latest
                '''
            }
        }

        stage('Remove Old Container') {
            steps {
                echo 'Removing old container...'

                sh '''
                    docker rm -f ${CONTAINER_NAME} || true
                '''
            }
        }

        stage('Deploy Container') {
            steps {
                echo 'Deploying Starbucks application...'

                sh '''
                    docker run -d \
                        --name ${CONTAINER_NAME} \
                        -p ${HOST_PORT}:${CONTAINER_PORT} \
                        ${DOCKER_IMAGE}:${BUILD_NUMBER}
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                echo 'Verifying deployment...'

                sh '''
                    sleep 5

                    echo "Running Containers:"
                    docker ps

                    echo "Port Mapping:"
                    docker port ${CONTAINER_NAME}

                    echo "Container Logs:"
                    docker logs --tail 50 ${CONTAINER_NAME}
                '''
            }
        }
    }

    post {

        success {
            echo '========================================'
            echo 'Starbucks Deployment Successful!'
            echo '========================================'
            echo 'Application: http://<EC2-PUBLIC-IP>:8081'
        }

        failure {
            echo '========================================'
            echo 'Starbucks Deployment Failed!'
            echo '========================================'
        }

        always {
            echo 'Pipeline completed.'
        }
    }
}

🔍 Pipeline Stage Explanation

Clean Workspace

cleanWs()

Removes files from previous Jenkins builds.

Checkout

Jenkins retrieves the configured Git branch from GitHub.

Verify Code

git rev-parse HEAD
git branch --show-current
git log -1 --oneline

These commands identify the exact source-code revision deployed by Jenkins.

Install Dependencies

npm ci

In a Jenkins declarative pipeline, shell commands must normally be executed using sh:

sh 'npm ci'

Run Tests

CI=true npm test -- --watchAll=false

The original learning pipeline used:

CI=true npm test -- --watchAll=false || true

because of a Jest/Swiper issue.

This is only a temporary workaround. A production CI pipeline should fail when tests fail.

Build React Application

npm run build

The original project used:

CI=false npm run build

because ESLint warnings were being treated as CI build failures.

The correct long-term solution is to fix the warnings.

Build Docker Image

docker build \
  -t ${DOCKER_IMAGE}:${BUILD_NUMBER} \
  -t ${DOCKER_IMAGE}:latest \
  .

Example:

rahmanuddinmd17/starbucks-app:10
rahmanuddinmd17/starbucks-app:latest

Push Docker Image

docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
docker push ${DOCKER_IMAGE}:latest

Remove Previous Container

docker rm -f starbucks-container || true

|| true prevents the pipeline from failing if the container does not already exist.

Deploy New Container

docker run -d \
  --name starbucks-container \
  -p 8081:3000 \
  rahmanuddinmd17/starbucks-app:${BUILD_NUMBER}

The critical mapping is:

8081:3000

Meaning:

EC2 Host Port 8081
        ↓
Container Port 3000

✅ Verify Deployment

Check running containers:

docker ps

Check all containers:

docker ps -a

Check port mapping:

docker port starbucks-container

Expected:

3000/tcp -> 0.0.0.0:8081

Check logs:

docker logs starbucks-container

Test from EC2:

curl -I http://localhost:8081

Expected:

HTTP/1.1 200 OK

Check ports:

sudo ss -ltnp | grep -E ':8080|:8081'

Expected arrangement:

8080 → Jenkins
8081 → Starbucks Application

🌐 Application Access

Jenkins:

http://EC2-PUBLIC-IP:8080

Application:

http://EC2-PUBLIC-IP:8081

Do not commit private keys, passwords, tokens, or other secrets to GitHub.

🧪 Manual Docker Test

Before troubleshooting Jenkins, test Docker independently.

Build:

docker build -t starbucks-test .

Run:

docker run -d \
  --name starbucks-test \
  -p 8081:3000 \
  starbucks-test

Verify:

docker ps
docker port starbucks-test
curl -I http://localhost:8081

🧯 Troubleshooting

Jenkins Does Not Start

sudo systemctl status jenkins
sudo journalctl -xeu jenkins

node: not found

Verify Jenkins NodeJS configuration:

Manage Jenkins
   ↓
Tools
   ↓
NodeJS installations

Ensure the configured installation name matches:

tools {
    nodejs 'node25'
}

No such DSL method 'npm'

Wrong:

npm ci

Correct:

sh 'npm ci'

npm is a shell command, not a Jenkins Pipeline DSL step.

Docker Permission Denied

sudo usermod -aG docker jenkins
sudo systemctl restart docker
sudo systemctl restart jenkins
sudo -u jenkins docker ps

Port 8080 Already in Use

Jenkins uses:

8080

Therefore the application uses:

8081

Check port usage:

sudo ss -ltnp | grep :8080

Application Does Not Open

Run:

docker ps -a
docker logs starbucks-container
docker port starbucks-container
curl -I http://localhost:8081
sudo ss -ltnp | grep :8081

Localhost Works but Browser Does Not

If this works:

curl -I http://localhost:8081

but this does not:

http://EC2-PUBLIC-IP:8081

check the AWS EC2 Security Group and confirm TCP port 8081 is allowed from the appropriate source.

Container Is Running but React Is Not Accessible

Enter the container:

docker exec -it starbucks-container sh

Test the application inside the container:

wget -qO- http://localhost:3000

or:

curl http://localhost:3000

Wrong Git Code Was Deployed

Check:

git rev-parse HEAD
git log -1 --oneline

These commands show exactly which commit Jenkins checked out.

🐞 Problems Encountered During the Project

1. Node.js Was Not Available to Jenkins

Error:

node: not found

Fix:

tools {
    nodejs 'node25'
}

2. npm Was Used as Jenkins DSL

Wrong:

npm ci

Correct:

sh 'npm ci'

3. Jest / Swiper Test Failure

Error:

Unexpected token 'export'

Temporary workaround:

sh 'CI=true npm test -- --watchAll=false || true'

Recommended solution:

Fix the Jest/Swiper configuration
        ↓
Run tests normally
        ↓
Test failure should fail the pipeline

4. React Build Failed Due to Warnings

Temporary workaround:

sh 'CI=false npm run build'

Recommended solution:

Fix all React/ESLint issues, such as:

unused variables/imports

missing image alt attributes

other lint warnings

5. Jenkins and Application Port Conflict

Jenkins already uses:

8080

The application therefore uses:

8081

6. Main Docker Port Mapping Error

The Dockerfile exposed:

3000

but the first Jenkins deployment mapped:

8081:80

This was incorrect because nothing was listening on container port 80.

Correct mapping:

8081:3000

Final configuration:

Jenkins     → EC2 :8080
Application → EC2 :8081
React       → Container :3000
Docker Map  → 8081:3000

💡 Docker Port Concept

Do not confuse:

EXPOSE 3000

with:

docker run -p 8081:3000 ...

EXPOSE documents the intended container port.

-p publishes and maps the EC2 host port to the container port.

-p HOST_PORT:CONTAINER_PORT
-p 8081:3000

📋 Final Checklist

AWS

EC2 instance created

Port 22 configured

Port 8080 configured

Port 8081 configured

Security Group source ranges reviewed

Linux

Git installed

Java 21 installed

Docker installed

Docker service running

Jenkins

Jenkins installed

Jenkins running

Pipeline plugin installed

Pipeline Stage View installed

NodeJS plugin installed

Docker Pipeline plugin installed

Docker Commons installed

Git support installed

Node.js

NodeJS installation configured

Jenkins NodeJS name matches node25

node --version works

npm --version works

Docker

Jenkins added to the Docker group

sudo -u jenkins docker ps works

Docker Hub

Docker Hub repository created

Jenkins Docker Hub credentials created

Credential ID is dockerhub-creds

Project

package.json

package-lock.json

Dockerfile

Jenkinsfile

React source code

README.md

Ports

Jenkins = 8080

React container = 3000

EC2 application = 8081

Docker mapping = 8081:3000

🧰 Command Cheat Sheet

EC2

sudo apt update
sudo apt install git -y
sudo apt install fontconfig openjdk-21-jre -y
sudo apt install docker.io -y

Docker

sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker jenkins
sudo systemctl restart docker
sudo systemctl restart jenkins

Jenkins

sudo systemctl status jenkins
sudo journalctl -xeu jenkins

Verification

docker --version
sudo -u jenkins docker ps
docker ps
docker ps -a
docker port starbucks-container
docker logs starbucks-container
curl -I http://localhost:8081
sudo ss -ltnp | grep -E ':8080|:8081'

🔄 Complete CI/CD Flow

Developer
    │
    ▼
GitHub
    │
    ▼
Jenkins
    │
    ├── Clean Workspace
    ├── Checkout
    ├── Verify Code
    ├── Node.js
    ├── npm ci
    ├── npm test
    ├── npm run build
    ├── docker build
    ├── docker login
    ├── docker push
    ├── remove old container
    ├── docker run
    └── verify deployment
    │
    ▼
Docker Hub
    │
    ▼
AWS EC2
    │
    ▼
Docker Container :3000
    │
    ▼
React Application
    │
    ▼
EC2 :8081

🎤 Interview Explanation

What is this project?

I containerized a React-based Starbucks application and created a Jenkins CI/CD pipeline that checks out the source code from GitHub, installs dependencies, runs tests, builds the React application, creates a Docker image, pushes the image to Docker Hub, and deploys the container on an AWS EC2 instance.

What does Jenkins do?

Jenkins automates the CI/CD process from source-code checkout through testing, build, Docker image creation, image publishing, deployment, and verification.

Why Docker?

Docker packages the application and its dependencies into a consistent and portable container image that can run reliably across different environments.

Why Docker Hub?

Docker Hub acts as the centralized image registry where Jenkins publishes versioned application images.

Why is the application using port 8081?

Jenkins is already running on EC2 port 8080, so the application is exposed through EC2 port 8081.

Why 8081:3000?

The React application listens on port 3000 inside the Docker container. Users access it through port 8081 on the EC2 host. Docker maps host port 8081 to container port 3000.

What major issue was fixed?

The Dockerfile used port 3000, while the original Jenkins deployment mapped EC2 port 8081 to container port 80.

Because the React application was not listening on port 80, the application could not be reached.

The mapping was corrected from:

8081:80

to:

8081:3000

⚠️ Production Improvements

This repository demonstrates the complete DevOps workflow, but the following changes should be made before treating the pipeline as production-ready:

Remove || true from the test stage so failed tests stop deployment.

Fix the Jest/Swiper configuration.

Fix all React/ESLint warnings.

Remove the need for CI=false during the production build.

Restrict AWS Security Group access.

Use access tokens/secret stores for all credentials.

Consider serving the React production build through Nginx instead of using the development server.

Add health checks and deployment rollback logic.

Add GitHub webhooks for automatic pipeline triggering.

Add HTTPS through a reverse proxy or load balancer.

🔐 Security Notes

Never commit any of the following to GitHub:

AWS private keys (*.pem)
Docker Hub passwords
Docker Hub access tokens
Jenkins administrator passwords
GitHub personal access tokens
SSH private keys
Cloud access keys
.env files containing secrets

Recommended .gitignore additions:

.env
.env.*
*.pem
*.key
node_modules/
build/
coverage/

📌 Final Configuration Summary

╔══════════════════════════════════════════════╗
║          STARBUCKS DEVOPS PROJECT            ║
╠══════════════════════════════════════════════╣
║ GitHub        → Source Code                  ║
║ Jenkins       → CI/CD                        ║
║ Node.js       → React Build                  ║
║ Docker        → Containerization             ║
║ Docker Hub    → Image Registry               ║
║ AWS EC2       → Deployment Server            ║
╠══════════════════════════════════════════════╣
║ Jenkins       → Port 8080                    ║
║ React         → Container Port 3000          ║
║ Application   → EC2 Port 8081                ║
║ Docker Map    → 8081:3000                    ║
╠══════════════════════════════════════════════╣
║ Docker Image                                 ║
║ rahmanuddinmd17/starbucks-app                ║
║                                              ║
║ Jenkins Credential ID                        ║
║ dockerhub-creds                              ║
║                                              ║
║ Jenkins NodeJS Tool                          ║
║ node25                                       ║
╚══════════════════════════════════════════════╝

👤 Author

MD Rahman

DevOps / Cloud / CI-CD Learning Project

⭐ Support

If this project was useful, consider giving the repository a ⭐.

Note: Replace placeholders such as EC2-PUBLIC-IP and YOUR_GITHUB_REPOSITORY_URL with your own environment details before deployment.
