Here is your **Jenkins Parameterized Branch Pipeline Setup Guide** formatted neatly in Markdown:

---

# Jenkins Parameterized Branch Pipeline Setup Guide

## 1. Install Required Plugins

First, install all necessary plugins in Jenkins:

1. Go to **Manage Jenkins** $\rightarrow$ **Plugins** $\rightarrow$ **Available plugins**.
2. Search and install:
* **SSH**
* **SSH Build Agents** *(required for launching agents via SSH)*
* **Gitea Plugin** *(VCS integration)*
* **Pipeline** *(workflow-aggregator)*


3. Restart Jenkins if prompted.

---

## 2. Agent Node Setup & Jenkins Registration

### Step 1: Prepare the Agent Server

On your Jenkins Agent server, run:

1. **Install OpenJDK:**
```bash
sudo apt update
sudo apt install -y openjdk-17-jdk

```


2. **Configure Passwordless SSH Access:**
* On the Jenkins Controller, view the SSH public key:
```bash
cat ~/.ssh/id_rsa.pub

```


* On the Jenkins Agent, append the copied key to `authorized_keys`:
```bash
mkdir -p ~/.ssh && chmod 700 ~/.ssh
echo "PASTE_CONTROLLER_PUBLIC_KEY_HERE" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys

```





### Step 2: Add Agent Node in Jenkins UI

1. Go to **Manage Jenkins** $\rightarrow$ **Nodes** $\rightarrow$ **New Node**.
2. Enter **Node Name** (e.g., `agent-node`), select **Permanent Agent**, and click **Create**.
3. Configure settings:
* **Remote root directory:** `/var/lib/jenkins` (or `/home/jenkins`)
* **Labels:** `agent-node`
* **Launch method:** Select *Launch agents via SSH*.
* **Host:** Enter the IP/hostname of the agent server.
* **Credentials:** Add/select the SSH private key credentials for the agent.


4. Click **Save** and verify the node launches and shows **In Service**.

---

## 3. Gitea Token Generation & Credentials Setup

### Step 1: Generate Access Token in Gitea

1. Log in to Gitea $\rightarrow$ **User Avatar** (top-right) $\rightarrow$ **Settings**.
2. Go to **Applications** $\rightarrow$ **Generate New Token**.
3. Set a **Token Name** (e.g., `jenkins-token`), grant repository permissions, and click **Generate Token**.
4. Copy the generated token string.

### Step 2: Add Gitea Credentials in Jenkins

1. Go to **Manage Jenkins** $\rightarrow$ **Credentials** $\rightarrow$ **System** $\rightarrow$ **Global credentials (unrestricted)**.
2. Click **Add Credentials**:
* **Kind:** Username with password
* **Username:** Your Gitea username
* **Password:** Paste the Gitea Access Token
* **ID:** `gittea-ap`


3. Click **Create**.

---

## 4. Add `Jenkinsfile` Across All Branches

> **Important Requirement:** When *Branch Specifier* is set to `*/${BRANCH}`, Jenkins looks for the `Jenkinsfile` on whatever branch is specified at trigger time. The `Jenkinsfile` must exist in the root directory of every branch (`master`, `feature`, etc.) you intend to build.

Create a `Jenkinsfile` in your repository with the following contents:

```groovy
pipeline {
    agent any
    
    stages {
        stage('Checkout & Merge') {
            steps {
                script {
                    echo "Target branch selected: ${params.BRANCH}"
                    sh """
                        git fetch --all
                        git checkout ${params.BRANCH} || git checkout -b ${params.BRANCH} origin/${params.BRANCH}
                        git pull origin ${params.BRANCH}
                    """
                }
            }
        }
    }
}

```

Commit and push this `Jenkinsfile` to **ALL** branches (`master` and `feature`).

---

## 5. Configure Parameterized Pipeline Job

1. Open your Jenkins job $\rightarrow$ **Configure**.
2. Check **This project is parameterized**.
3. Click **Add Parameter** $\rightarrow$ **String Parameter**:
* **Name:** `BRANCH`
* **Default Value:** `master`
* **Description:** Target branch to checkout or merge.


4. Scroll down to **Pipeline**:
* **Definition:** Pipeline script from SCM
* **SCM:** Git
* **Repository URL:** Enter your Gitea repository URL.
* **Credentials:** Select `gittea-ap`.
* **Branch Specifier:** `*/${BRANCH}` *(Or set to `*/master` if you prefer Jenkins to always fetch the script from master regardless of the target branch)*
* **Script Path:** `Jenkinsfile`


5. Click **Save**.

---

## 6. Testing & Verification

### Test Case 1: Build `master` Branch

1. Click **Build with Parameters**.
2. Set `BRANCH = master` and click **Build**.
3. Check **Console Output** to verify `master` was checked out successfully.

### Test Case 2: Build `feature` Branch

1. Click **Build with Parameters**.
2. Set `BRANCH = feature` and click **Build**.
3. Check **Console Output** to confirm:
* Jenkins successfully locates `Jenkinsfile` on the `feature` branch.
* Output displays: `Target branch selected: feature` and `Switched to a new branch 'feature'`.
* Pipeline completes with `Finished: SUCCESS`.
