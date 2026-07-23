Step-by-Step Configuration
Step 1: Create the Job xfusion-app-deployment
Go to Jenkins Dashboard > New Item.

Name the job xfusion-app-deployment.

Select Pipeline (or Freestyle project) and click OK.

Step 2: Configure the Job
Option A: If using Pipeline (Jenkinsfile in your Gitea Repo)
Under Build Triggers, check Poll SCM and set the schedule to * * * * * (polls every minute).

Under Pipeline:

Definition: Pipeline script from SCM

SCM: Git

Repository URL: [https://3000-port-bzdfjydvhhy7hmh3.labs.kodekloud.com/sarah/web.git](https://3000-port-bzdfjydvhhy7hmh3.labs.kodekloud.com/sarah/web.git)

Branch Specifier: */master

Script Path: Jenkinsfile

Update your Jenkinsfile in your Git repository to run git pull on stapp01:

```Groovy
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
```
Option B: If using Freestyle Project
Under Source Code Management:

Select Git.

Repository URL: [https://3000-port-bzdfjydvhhy7hmh3.labs.kodekloud.com/sarah/web.git](https://3000-port-bzdfjydvhhy7hmh3.labs.kodekloud.com/sarah/web.git)

Branch Specifier: */master

Under Build Triggers:

Check Poll SCM and set the schedule to * * * * *.

Under Build Steps:

Add step: Execute shell

Command:

Bash
ssh -o StrictHostKeyChecking=no sarah@stapp01 "cd /var/www/html && git pull origin master"
Step 3: Save & Test
Save the job configuration.

Push a new commit to master in Gitea.

Within 1 minute, Poll SCM will trigger xfusion-app-deployment, which executes cd /var/www/html && git pull origin master on App Server 1 to update the live Apache files.

---
Job 2 (manage-services):

Step 1: Create the Job
Go to the Jenkins Dashboard and click New Item on the left menu.

Enter the Item Name: manage-services.

Select Freestyle project (or Pipeline) and click OK.

Step 2: Configure the Upstream Dependency (Build Trigger)
In the job configuration page, scroll down to the Build Triggers section.

Check the box for Build after other projects are built.

In the Projects to watch text field, enter:

Plaintext
xfusion-app-deployment
Select the option Trigger only if build is stable.

Step 3: Configure the Apache Restart Command
Option A: If you created a Freestyle Project
Scroll down to the Build Steps section.

Click Add build step and select Execute shell.

Enter the SSH command to restart the httpd service on App Server 1:

Bash
ssh -o StrictHostKeyChecking=no sarah@stapp01 "sudo systemctl restart httpd"
Option B: If you created a Pipeline Project
Scroll down to the Pipeline section.

In the Script block, paste:

```Groovy
pipeline {
    agent any
    stages {
        stage('Restart Service') {
            steps {
                sh 'ssh -o StrictHostKeyChecking=no sarah@stapp01 "sudo systemctl restart httpd"'
            }
        }
    }
}
```
Step 4: Save & Verify
Click Save at the bottom.

To test the pipeline flow:

Make a commit and push to master in your repository.

xfusion-app-deployment will run first, performing git pull on /var/www/html.

As soon as xfusion-app-deployment finishes successfully (Stable), manage-services will automatically start and restart httpd on App Server 1.
