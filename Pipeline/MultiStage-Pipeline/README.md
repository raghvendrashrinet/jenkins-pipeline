# 🚀 Node.js User Service - CI/CD Multistage Pipeline

This repository contains a sample Node.js microservice (`user-service`) configured with an automated **Jenkins Multistage Declarative Pipeline** for Continuous Integration and Continuous Deployment (CI/CD).

---

## 📋 Table of Contents
- [Pipeline Architecture](#-pipeline-architecture)
- [Pipeline Stages Overview](#-pipeline-stages-overview)
- [Prerequisites](#-prerequisites)
- [Environment Variables & Secrets](#-environment-variables--secrets)
- [Jenkinsfile Code](#-jenkinsfile-code)
- [Local Setup & Testing](#-local-setup--testing)

---

## 🏗 Pipeline Architecture
```
[ Git  Push / Poll SCM ]
       │
       ▼
┌───────────┐
│ Checkout  │
└─────┬─────┘
      │
      ▼
┌─────────────┐
│ Code Lint   │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ Unit Tests  │
└──────┬──────┘
       │
       ▼
┌───────────────┐
│ Build Docker  │
└───────┬───────┘
        │
        ▼
┌─────────────────┐
│ Security Scan   │
└────────┬────────┘
         │
         ▼
┌──────────────────┐
│ Deploy Staging   │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ Deploy Prod      │  (Requires Manual Approval)
└──────────────────┘

```
---

## ⚙️ Pipeline Stages Overview

1. **Checkout SCM**: Fetches the latest source code from the target Git branch.
2. **Lint & Static Analysis**: Runs `npm run lint` to enforce code quality and styling guidelines.
3. **Run Unit Tests**: Executes `npm test` and publishes test reports (e.g., JUnit XML results).
4. **Build & Tag Docker Image**: Containerizes the application using the `Dockerfile` and tags it with the short Git commit SHA.
5. **Security Vulnerability Scan**: Scans the compiled Docker image for OS and package vulnerabilities using **Trivy**.
6. **Deploy to Staging**: Automatically deploys the containerized application to the Staging server.
7. **Production Approval & Deployment**: Pauses the pipeline for manual approval before deploying to the Production environment.

---

## 🛠 Prerequisites

Before running this pipeline in Jenkins, ensure the following are installed and configured:

### Jenkins Plugins Needed
* **Git Plugin**
* **Pipeline** (Declarative Pipeline)
* **Docker Pipeline Plugin**
* **Credentials Plugin**

### System Requirements on Jenkins Agent
* **Node.js** v18+ / `npm`
* **Docker Engine** running and accessible by the `jenkins` user
* **Trivy CLI** (or run Trivy via Docker container inside the pipeline step)

---

## 🔐 Environment Variables & Credentials

Set up the following global credentials in **Jenkins Dashboard** > **Manage Jenkins** > **Credentials**:

| Credential ID | Kind | Description |
| :--- | :--- | :--- |
| `docker-hub-credentials` | Username with password | Authentication for Docker Registry |
| `staging-ssh-key` | SSH Username with private key | Deployment access to Staging server |
| `prod-ssh-key` | SSH Username with private key | Deployment access to Production server |

---

## 📄 Jenkinsfile

Here is the declarative pipeline configuration (`Jenkinsfile`) stored at the root of the project:

```groovy
pipeline {
    agent any

    environment {
        APP_NAME    = 'user-service'
        REGISTRY    = 'your-dockerhub-username'
        IMAGE_TAG   = "${env.BUILD_NUMBER}-${BUILD_TIMESTAMP}"
        STAGING_IP  = '10.0.1.50'
        PROD_IP     = '10.0.2.50'
    }

    options {
        timeout(time: 1, unit: 'HOURS')
        disableConcurrentBuilds()
        ansiColor('xterm')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Code Quality & Lint') {
            steps {
                sh 'npm install'
                sh 'npm run lint'
            }
        }

        stage('Unit Tests') {
            steps {
                sh 'npm test'
            }
            post {
                always {
                    junit allowEmptyResults: true, testResults: '**/test-results.xml'
                }
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                script {
                    docker.withRegistry("[https://index.docker.io/v1/](https://index.docker.io/v1/)", 'docker-hub-credentials') {
                        def appImage = docker.build("${REGISTRY}/${APP_NAME}:${IMAGE_TAG}")
                        appImage.push()
                        appImage.push("latest")
                    }
                }
            }
        }

        stage('Security Scan (Trivy)') {
            steps {
                sh "trivy image --severity HIGH,CRITICAL ${REGISTRY}/${APP_NAME}:${IMAGE_TAG}"
            }
        }

        stage('Deploy to Staging') {
            steps {
                sshagent(['staging-ssh-key']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no deploy@${STAGING_IP} "
                            docker pull ${REGISTRY}/${APP_NAME}:${IMAGE_TAG} && \
                            docker stop ${APP_NAME} || true && \
                            docker rm ${APP_NAME} || true && \
                            docker run -d --name ${APP_NAME} -p 8080:8080${REGISTRY}/${APP_NAME}:${IMAGE_TAG}
                        "
                    """
                }
            }
        }

        stage('Promote to Production') {
            steps {
                input message: 'Approve deployment to Production?', ok: 'Deploy'
                sshagent(['prod-ssh-key']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no deploy@${PROD_IP} "
                            docker pull ${REGISTRY}/${APP_NAME}:${IMAGE_TAG} && \
                            docker stop ${APP_NAME} || true && \
                            docker rm ${APP_NAME} || true && \
                            docker run -d --name ${APP_NAME} -p 8080:8080${REGISTRY}/${APP_NAME}:${IMAGE_TAG}
                        "
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline succeeded! App version ${IMAGE_TAG} deployed successfully."
        }
        failure {
            echo "Pipeline failed. Check stage logs for troubleshooting."
        }
    }
}
```
####🧪 Local Setup & Testing
To run and test the application locally before committing:

#### Clone the repository:

```Bash
git clone [https://github.com/your-username/user-service.git](https://github.com/your-username/user-service.git)
cd user-service
Install dependencies and run tests:
```
```Bash
npm install
npm test
Build the Docker container locally:
```

Bash
docker build -t user-service:local .
docker run -p 8080:8080 user-service:local
