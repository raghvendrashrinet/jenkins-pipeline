# Exercise: Automated Deployment & Service Management in Jenkins

## 📌 1. Objective

Set up an automated CI/CD pipeline using Jenkins to:
* **Pull source code** from a Git repository (`sarah/web.git`).
* **Deploy application files** to App Server 1 (`stapp01`) under `/var/www/html`.
* **Trigger a downstream job** (`manage-services`) that restarts the Apache web server (`httpd`) on App Server 1.

---

## 🏗️ 2. Architecture & Setup Overview

| Component | Description |
| :--- | :--- |
| **Jenkins Server** | Runs the CI/CD pipelines. |
| **App Server 1 (`stapp01`)** | Target deployment server. |
| **Deployment User** | `sarah` |
| **Target Web Directory** | `/var/www/html` |
| **Web Service** | `httpd` (Apache) |

---

## 🛠️ 3. Issues Encountered & Root Cause Analysis

| # | Issue / Error Log | Root Cause | Solution Applied |
| :-: | :--- | :--- | :--- |
| **1** | `can't cd to /var/www/html` | Pipeline was executing locally on the Jenkins server instead of target App Server 1. | Modified pipeline to execute deployment steps over SSH to `stapp01`. |
| **2** | `NoSuchMethodError: No such DSL method 'steps'` | Missing top-level `pipeline { ... }` block required for Declarative Pipeline syntax. | Wrapped the stage definitions inside a complete `pipeline { ... }` block. |
| **3** | `Could not resolve hostname app-server-1` | `app-server-1` alias was not resolved by DNS on the Jenkins host. | Updated the connection string to use the environment's recognized hostname: `stapp01`. |
| **4** | `sudo: a password is required` | User `sarah` lacked passwordless sudo privileges for `systemctl restart httpd` on `stapp01`. | Added a `NOPASSWD` entry for `sarah` in `/etc/sudoers` on App Server 1. |

---

## 🚀 4. Step-by-Step Implementation & Solution

### Step 1: Configure Passwordless Sudo on App Server 1
To allow Jenkins to restart the web server non-interactively via SSH, grant `sarah` passwordless sudo permissions on App Server 1 (`stapp01`).

1. **SSH into App Server 1:**
   ```bash
   ssh sarah@stapp01
   ```

2.Open the sudoers configuration file:

```Bash
sudo visudo
```
Add the following line at the end of the file:

```Plaintext
sarah ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart httpd
Save and exit (changes take effect immediately).
```
---
Step 2: Configure Upstream Job (xfusion-app-deployment)Update the Jenkinsfile in the application repository (sarah/web.git):

```Groovy
pipeline {
    agent any
    stages {
        stage('Deploy to App Server 1') {
            steps {
                sh '''
                    ssh -o StrictHostKeyChecking=no sarah@stapp01 "cd /var/www/html && git pull origin master"
                '''
            }
        }
    }
    post {
        success {
            // Trigger downstream job upon successful deployment
            build job: 'manage-services', wait: false
        }
    }
}
```
Step 3: Configure Downstream Job (manage-services)Update the pipeline configuration for manage-services to execute the remote service restart:
```Groovy
pipeline {
    agent any
    stages {
        stage('Restart Apache Web Service') {
            steps {
                sh '''
                    ssh -o StrictHostKeyChecking=no sarah@stapp01 "sudo systemctl restart httpd"
                '''
            }
        }
    }
}
```
✅ 5. Final Verification
1.Trigger xfusion-app-deployment:
- Jenkins connects to stapp01 via SSH as user sarah.
- Navigates to /var/www/html and pulls the latest code from master.
- Pipeline completes successfully (Finished: SUCCESS).
2. Automated Trigger of manage-services:
  - Upstream completion triggers manage-services.
  - Executes sudo systemctl restart httpd on stapp01 without prompting for a password.
  - Pipeline completes successfully (Finished: SUCCESS).
