# Jenkins Job Configuration Guide

## Step-by-Step Configuration

---

### Step 1: Create the Job `xfusion-app-deployment`

1. Go to **Jenkins Dashboard** > **New Item**.
2. Name the job: `xfusion-app-deployment`.
3. Select **Pipeline** (or **Freestyle project**) and click **OK**.

---

### Step 2: Configure the Job

Choose **Option A** or **Option B** depending on your project type.

#### **Option A: Using Pipeline (Jenkinsfile in Gitea Repo)**

1. **Build Triggers**:
   * Check **Poll SCM**.
   * Set Schedule to: `* * * * *` *(polls every minute)*.

2. **Pipeline Configuration**:
   * **Definition**: `Pipeline script from SCM`
   * **SCM**: `Git`
   * **Repository URL**: `https://3000-port-bzdfjydvhhy7hmh3.labs.kodekloud.com/sarah/web.git`
   * **Branch Specifier**: `*/master`
   * **Script Path**: `Jenkinsfile`

3. **Jenkinsfile Example**:
   Update your `Jenkinsfile` in your Git repository to run `git pull` on `stapp01`:

   ```groovy
   pipeline {
       agent any
       stages {
           stage('Deploy') {
               steps {
                   sh 'ssh -o StrictHostKeyChecking=no sarah@stapp01 "cd /var/www/html && git pull origin master"'
               }
           }
       }
   }


   ---
   Option B: Using Freestyle Project
1. Source Code Management:

Select Git.

Repository URL: https://3000-port-bzdfjydvhhy7hmh3.labs.kodekloud.com/sarah/web.git

Branch Specifier: */master

2. Build Triggers:

Check Poll SCM.

Set Schedule to: * * * * *

3. Build Steps:

Add build step: Execute shell

Command:
```bash
ssh -o StrictHostKeyChecking=no sarah@stapp01 "cd /var/www/html && git pull origin master"
```
