## Jenkins Controller-Agent architecture

Jenkins, the concept of a "runner" is handled differently through Agents (formerly called Slaves or Nodes
- Controller: The main server that manages the UI, schedules jobs, and monitors agents.\
- Agent: The machine (physical, virtual, or container) that actually executes the build steps (the equivalent of a GitHub Runner).

#### Setup Agent
1. Static Agents (Traditional Setup)
   you manually provision machines (VMs or physical servers) and connect them to the Jenkins Controller permanently
   * Setup: You install Java on the target machine and configure it to connect to the controller via SSH, JNLP (Java Web Start), or by launching it as a system service.
   *  Labeling: Similar to GitHub's ubuntu-latest, you assign Labels to these agents (e.g., linux, docker, ubuntu-22). 
   * Usage: In your Jenkinsfile, you specify the agent using the label:
```jenkinsfile
  pipeline {
      agent { label 'ubuntu-22' }
      stages {
          stage('Build') {
              steps {
                  sh 'echo "Running on a persistent Ubuntu agent"'
              }
          }
      }
  }   
```
Difference: Unlike GitHub Actions, these agents are usually long-lived. You are responsible for maintaining the OS, cleaning up workspace files, and ensuring tools (like Maven, Node, Docker) are installed.

#### 2. Ephemeral Agents (Cloud-Native Approach)
To mimic the GitHub Actions model (where runners are created for a single job and then destroyed), Jenkins uses Ephemeral Agents.  This is often achieved using plugins like Kubernetes, Docker, or EC2.

---

### 1. Prepare the Agent Machine (Commands)
Run these commands on the new agent machine (e.g., Ubuntu) to install Java, create a user, and set up SSH access.

- Install Java (Required): Jenkins agents require Java. Match the version to your controller (usually Java 11 or 17).
```
sudo apt update
sudo apt install openjdk-17-jre-headless -y
java -version # Verify installation
```
- Create a Jenkins User (Optional but Recommended):
```
sudo useradd -m -s /bin/bash jenkins
sudo passwd jenkins # Set a password
```
Setup SSH Keys: You need to allow the Jenkins Controller to SSH into this machine without a password. Option A: If you already have a public key from the Controller:

##### On the AGENT machine
```
sudo mkdir -p /home/jenkins/.ssh
echo "PASTE_CONTROLLER_PUBLIC_KEY_HERE" | sudo tee -a /home/jenkins/.ssh/authorized_keys
sudo chown -R jenkins:jenkins /home/jenkins/.ssh
sudo chmod 700 /home/jenkins/.ssh
sudo chmod 600 /home/jenkins/.ssh/authorized_keys
```
#### Option B: Generate a new key pair on the Controller and copy it:
```
# On the CONTROLLER machine
ssh-keygen -t rsa -b 4096 -f jenkins_agent_key
# Copy the public key to the agent
ssh-copy-id -i jenkins_agent_key.pub jenkins@<AGENT_IP_ADDRESS>
```
Install Build Tools: Since this is a persistent agent, install tools globally:
```
sudo apt install git docker.io maven nodejs -y
sudo usermod -aG docker jenkins # Allow jenkins user to run docker
```

### 2. Configure Jenkins Controller (UI Steps)
No commands are needed here; perform these steps in the Jenkins web interface.

##### Add Credentials:
`Go to Manage Jenkins > Credentials > (Global) > Add Credentials.`
```
Kind: SSH Username with private key.
Username: jenkins (or the user created in Step 1).
Private Key: Paste the private key content (from jenkins_agent_key if you generated one in Option B above) or select "From the Jenkins master ~/.ssh".
ID: agent-ssh-key. Click Create.
```
##### Install the Plugin:
Go to Manage Jenkins > Plugins (or Manage Plugins).
Click the Available plugins tab.
Search for `SSH Build Agents.`
Check the box and click Install.


- Create the Node:
```
Go to Manage Jenkins > Nodes > New Node.
Node name: ubuntu-runner-01.
Select Permanent Agent > Create.
Configure Node Details:
Remote root directory: /home/jenkins (Must match the user's home dir).
Labels: ubuntu-self-hosted (Use this in your Jenkinsfile).
Usage: Use this node as much as possible.
Launch method: Launch agents via SSH.
Host: <AGENT_IP_ADDRESS>.
Credentials: Select agent-ssh-key.
Host Key Verification: Non verifying Verification Strategy (for quick setup).
Click Save.
```
- Jenkins node configuration SSH launch

View all
3. Verify Connection
Jenkins will automatically attempt to connect.

If successful, the node status turns Green (Online).
If it fails, check the log in the node page or verify SSH access manually from the controller:
# Run on CONTROLLER to test connection
ssh -i <path_to_private_key> jenkins@<AGENT_IP_ADDRESS>

4. Usage in Pipeline
Use the label defined in Step 3 in your Jenkinsfile:
```Jenkinsfile
pipeline {
    agent { label 'ubuntu-self-hosted' }
    stages {
        stage('Build') {
            steps {
                sh 'java -version'
                sh 'git --version'
            }
        }
    }
}
```
---

#### To set up a Jenkins pipeline with source code hosted on Gitea, follow these steps to install the plugin, configure credentials, and create the pipeline
* 1. Install the Gitea Plugin  
   The Gitea integration requires a specific plugin.
```
Go to Manage Jenkins > Plugins > Available plugins. 
Search for Gitea.
Install the Gitea plugin (and restart if prompted)
```


* 2. Generate Gitea Access Token
 Jenkins needs a token to access your repositories and manage webhooks.
```
Log in to Gitea.
Go to Settings (click your avatar) > Applications.
Under "Manage Access Tokens", create a new token:
Name: jenkins-ci
Permissions: Ensure Read access to repositories (and Write if you want Jenkins to update build statuses). 
Copy the token immediately (you cannot see it again)
```

* 3. Add Credentials to Jenkins
```
Go to Manage Jenkins > Credentials > (Global) > Add Credentials. 
Kind: Select Gitea Personal Access Token (if available) or Secret text. 
Token: Paste the token copied from Gitea.
ID: gitea-token (remember this ID).
Click Create.
```

* 4. Configure Gitea Server in Jenkins
```
 Go to Manage Jenkins > Configure System. 
Scroll to the Gitea Servers section. 
Click Add Gitea Server:
Name: my-gitea
Server URL: Your Gitea URL (e.g., https://gitea.example.com).
Credentials: Select gitea-token.
Manage Hooks: Check this box to let Jenkins automatically create webhooks in Gitea. 
Click Save.
```

---

### 5. Create the Pipeline
You have two options depending on your needs:

##### Option A: Multibranch Pipeline (Recommended)
Automatically discovers branches and creates jobs for those containing a Jenkinsfile.
```
Click New Item > Enter name (e.g., my-app) > Select Multibranch Pipeline > OK. 
Branch Sources:
Click Add source > Gitea.
Server: Select my-gitea.
Owner: Your Gitea username or organization name.
Credentials: Select gitea-token.
Discover branches: Ensure "All branches" is selected.
Click Save. Jenkins will scan Gitea and build any branch with a Jenkinsfile. 
```

##### Option B: Standard Pipeline
For a single specific branch/repository.
```
Click New Item > Enter name > Select Pipeline > OK. 
Pipeline section:
Definition: Pipeline script from SCM.
SCM: Select Git.
Repository URL: https://gitea.example.com/username/repo.git.
Credentials: Select gitea-token (or an SSH key if you configured SSH).
Branch Specifier: */main (or your branch name).
Build Triggers:
Check Poll SCM (leave schedule empty) OR rely on the webhook configured in Step 4. 
Click Save.
```
#### manual pipeline script execution namde deploy on added agent

Pipeline Script
```
pipeline {
    agent {
        label 'stapp01'
    }
    stages {
        stage('Deploy') {
            steps {
                sh '''
                    cd /var/www/html
                    git pull origin main || git pull origin master
                '''
            }
        }
    }
}
```

Test > Build Now 

##### 6. Verify Webhook
If you enabled "Manage Hooks" in Step 4, Jenkins automatically created the webhook.

Go to your repository in Gitea > Settings > Webhooks. 
You should see a webhook pointing to https://<jenkins-url>/gitea-webhook/post.
Push a commit to your repository; Jenkins should automatically trigger a build. 

Example Jenkinsfile
```Jenkinsfile
pipeline {
    agent { label 'ubuntu-self-hosted' } // Uses the agent you created earlier
    stages {
        stage('Checkout') {
            steps {
                echo 'Code checked out from Gitea'
            }
        }
        stage('Build') {
            steps {
                sh 'echo "Building..."'
                // Add your build commands here
            }
        }
    }
}
```

You must manually create this file in your Gitea repository. It is a text file that defines your pipeline logic (stages, steps, agents) and must be committed to your source code.   

How to Create It -> In Gitea UI: -> Navigate to your repository.
```
Click Add File > New File.
Name the file exactly Jenkinsfile (case-sensitive).
Paste your pipeline code (like the example provided earlier).
Click Commit Changes.
Via Command Line:
# Clone your repo
git clone https://gitea.example.com/username/repo.git
cd repo
```
# Create the file
nano Jenkinsfile
# (Paste your pipeline code here and save)

# Commit and push
git add Jenkinsfile
git commit -m "Add Jenkins pipeline"
git push

What Is Automatic?
Jenkins Discovery: Once you push the Jenkinsfile to a branch, Jenkins (configured as a Multibranch Pipeline) will automatically detect the file, create a job for that branch, and trigger the first build.
Webhooks: If configured, Gitea will automatically notify Jenkins of the new commit.
