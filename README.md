# 🏗️ Jenkins Knowledge Base & Pipeline Library

Welcome to the central Knowledge Base (KB) for all things Jenkins. This repository serves as a single source of truth for CI/CD pipeline templates, shared library documentation, troubleshooting guides, and infrastructure configuration best practices.

---

## 📁 Repository Structure

```text
├── .github/               # GitHub issue templates & workflows
├── pipelines/             # Standardized Declarative/Scripted Jenkinsfiles
│   ├── java-maven.groovy
│   ├── nodejs-react.groovy
│   └── python-docker.groovy
├── vars/                  # Custom steps for Jenkins Shared Libraries
├── docs/                  # In-depth guides and troubleshooting
│   ├── error-solutions.md
│   ├── agent-setup.md
│   └── security-best-practices.md
└── README.md              # This file

## 🏗️ Jenkins Architecture Overview

Jenkins uses a **Controller-Agent (formerly Master-Agent) architecture** to manage workloads, scale builds distributedly, and ensure high availability across environments.

```text
    +-----------------------------------------------+
    |               Jenkins Controller              |
    |  (UI, Configuration, Plugins, Scheduling)    |
    +-----------------------+-----------------------+
                            |
           +----------------+----------------+
           |                                 |
           v                                 v
+------------------------+        +-------------------------+
|    Agent Node 1        |        |    Agent Node 2         |
| (Linux/Docker Executor)|        | (Windows/macOS Executor)|
+------------------------+        +-------------------------+
```
## 🔧 Setup & Installation Requirements

To successfully provision the Jenkins architecture, distinct components must be installed depending on the node's role.

### 🧠 1. Jenkins Controller (Master Node) Setup
The Controller acts as the orchestrator. It requires the core Jenkins service along with a Java runtime environment to execute.

*   **Java Development Kit (JDK):** Jenkins is a Java application. It requires **Java 11 or Java 17** (depending on your Jenkins version LTS). *OpenJDK* is highly recommended.
*   **Jenkins Core Package:** The actual `jenkins.war` file or native OS package manager setup (`apt-get`, `yum`, or a standard Docker container image `jenkins/jenkins:lts`).
*   **Git:** Essential for the controller to perform initial lightweight polling, indexing, and fetching of Jenkinsfile pipelines from source repositories.
*   **SSL/TLS Certificates:** Required to secure the Web UI via HTTPS (often handled via a reverse proxy like Nginx or an AWS ALB placed in front of the Controller).

---

### 💪 2. Jenkins Agent (Worker Node) Setup
Agents require minimal Jenkins overhead but must be heavily configured with the **actual tools your software needs to compile and run**.

#### Core Agent Software (Mandatory)
*   **Java Runtime Environment (JRE/JDK):** The agent needs Java (matching the Controller's version) to run the `agent.jar` client binary that communicates back to the Controller.
*   **Remoting Binary (`agent.jar`):** Automatically sent by the Controller during the SSH handshake or downloaded manually if connecting via Inbound TCP (JNLP).
*   **Git:** Necessary to clone the application source code into the local executor workspace.

#### Build Tools & Runtimes (Depends on your tech stack)
Unlike the Controller, the Agent needs full access to your build toolchains. Common installations include:

| Category | Software / Tools to Install |
| :--- | :--- |
| **Containers** | Docker Engine, `containerd`, `kubectl`, Helm (for building images & deployment) |
| **Java Stacks** | Maven, Gradle |
| **Web / JS** | Node.js, NPM, Yarn |
| **Python** | Python3, `pip`, `virtualenv` |
| **Compilers** | GCC, Make, Go Lang, .NET SDK |

---

## 🏗️ Step-by-Step Installation Flow

### Step 1: Prepare the Controller
1. Install Java: `sudo apt install openjdk-17-jdk`
2. Add the Jenkins repository keys and install the service.
3. Adjust the system firewall to allow traffic on port `8080` (Web UI) and port `50000` (Agent inbound communication).

### Step 2: Prepare the Agent
1. Provision a clean machine or VM.
2. Install Java: `sudo apt install openjdk-17-jre-headless`
3. Create a dedicated Jenkins deployment user:
   ```bash
   sudo useradd -m -d /home/jenkins -s /bin/bash jenkins
