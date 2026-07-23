

1. - Create a Jenkins job named devops-app-deployment and configure it so that if anyone pushes any new change to the origin repository in master branch, the job should auto build and deploy the latest code on App Server 1 under /var/www/html directory.
Before deployment, ensure that the ownership of the /var/www/html directory is set to user sarah, so that Jenkins can successfully deploy files to that directory.


2. - SSH into App Server 1 using sarah user credentials mentioned above. Under sarah user's home (/home/sarah/web) you will find a cloned Git repository named web. Under this repository there is an index.html file, update its content to Welcome to the xFusionCorp Industries, then push the changes to the origin into master branch. This push must trigger your Jenkins job and the latest changes must be deployed on the server,
4. - also make sure it deploys the entire repository content not only index.html file.
---
### Solution
#### Step 1: Install Required Jenkins Plugins
Go to Manage Jenkins > Plugins > Available plugins.

Search for and install the following plugins:
- Gitea Plugin (for integration and webhook/polling support)
- Credentials Plugin (to securely store tokens and SSH keys)
- Pipeline (if not already installed)
- SSH
- SSH Build Agent (if you want to add another node)

Select Install without restart (or restart Jenkins if prompted).

#### Step 2: Generate Gitea Access Token
Log in to your Gitea web interface as user sarah.
- Go to User Settings (top-right avatar) > Applications.
- Under Generate New Token:
   Token Name: jenkins-token -> Click Generate Token.

- Copy the generated token

#### Step 3: Add Gitea Token to Jenkins Credentials
Go to Manage Jenkins > Credentials > System > Global credentials (unrestricted).
```
Click Add Credentials. - > Fill in the details:
Kind: Secret text
Secret: Paste your Gitea Access Token here.
ID: gitea-token (or leave blank to auto-generate)
```

Click Create.

#### Step 4: Add SSH / Password Credentials for Target App Server
To allow Jenkins to copy files to App Server 1 (stapp01):

- Go to Manage Jenkins > Credentials > System > Global credentials.

Click Add Credentials.
```
Fill in the details:

Kind: Username with password (or SSH Username with private key if using key pair)

Username: sarah (or stapp01)

Password: Passphrase/password for the user

ID: app-server-creds

Click Create.
```
#### Step 5: Configure Gitea in Jenkins System Settings
Go to Manage Jenkins > System (or Configure System).

- Scroll down to the Gitea Servers section.

- Click Add Gitea Server:
```
Name: Gitea

Server URL: [https://3000-port-bzdfjydvhhy7hmh3.labs.kodekloud.com](https://3000-port-bzdfjydvhhy7hmh3.labs.kodekloud.com) (or your Gitea domain)

Credentials: Select the Gitea Token for Sarah credential created in Step 3.

Click Test Connection to verify.
```
Click Save.

#### Step 6: Create the Jenkinsfile in Code Repository
In your local code repository for project web:

Create a file named Jenkinsfile at the repository root:

```Groovy
pipeline {
    agent any

    stages {
        stage('Deploy') {
            steps {
                // Deploys files to App Server 1 (/var/www/html)
                sh 'scp -o StrictHostKeyChecking=no -r * sarah@stapp01:/var/www/html/'
            }
        }
    }
}
```
Commit and push it to master:

```Bash
git add Jenkinsfile
git commit -m "Add deployment Jenkinsfile"
git push origin master
```
#### Step 7: Create and Configure the Jenkins Pipeline Job
- Go to Jenkins Dashboard > New Item.
- Enter the Item Name: devops-app-deployment.
- Select Pipeline and click OK.

Build Triggers:
 - Check Poll SCM.
 - Set Schedule to * * * * * (checks for new pushes every minute).

Pipeline Section:
 - Definition: Pipeline script from SCM
 - SCM: Git
 - Repository URL: [https://3000-port-bzdfjydvhhy7hmh3.labs.kodekloud.com/sarah/web.git](https://3000-port-bzdfjydvhhy7hmh3.labs.kodekloud.com/sarah/web.git)
 - Credentials: Select your Gitea token or user credentials if private.
 - Branch Specifier: */master
 - Script Path: Jenkinsfile

Click Save.

#### Step 8: Verify Auto Trigger & Deployment
Make a code change in your repo and push to master:

```Bash
echo "<h1>Deployment Success</h1>" > index.html
git add index.html
git commit -m "Test trigger"
git push origin master
```
Wait 1 minute.

Jenkins will automatically detect the new commit hash, trigger a build, and deploy the code to /var/www/html/ on App Server 1.
