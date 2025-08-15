# Portfolio Website Deployment using Jenkins Pipeline

This guide walks you through deploying a static portfolio website to an Apache server using Jenkins. It’s beginner-friendly and includes optional GitHub webhook automation.

---

## Prerequisites

### 1. Infrastructure

* **Two servers**:

  * **Jenkins Server** (runs Jenkins and pipeline)
  * **Apache Web Server** (serves your portfolio website)
* Both launched using the same `.pem` key (or configured SSH credentials).

### 2. Software Requirements

| Component         | Jenkins Server | Apache Server |
| ----------------- | -------------- | ------------- |
| Java (OpenJDK 17) | ✅ Required     | ❌ Not needed  |
| Apache2           | ❌ Not needed   | ✅ Required    |
| Git               | ✅ Required     | ✅ Required    |
| Jenkins           | ✅ Required     | ❌ Not needed  |

**Install Apache2 on the Web Server**:

```bash
sudo apt update
sudo apt install -y apache2
sudo systemctl enable apache2
sudo systemctl start apache2
```

---
**Changing Ownership of /var/www/html directory from root to ubuntu user**

```bash
    sudo chown -R ubuntu:ubuntu /var/www/html/
```
![](/portfolio-app-img/ownership-change.png)
---

## Setting Up Jenkins Credentials

We are using SSH credentials to connect to the Apache server.

**Step 1 — Copy PEM Key to Jenkins Server**:

```bash
scp -i pem-key-server.pem pem-key-server.pem ubuntu@<JENKINS_SERVER_PUBLIC_IP>:/home/ubuntu/
```

**Step 2 — Add PEM Key to Jenkins Credentials**:

* Go to: **Manage Jenkins → Credentials → (global) → Add Credentials**
* Kind: **SSH Username with private key**
* Username: `ubuntu`
* Private Key: Enter directly → paste PEM file contents
* ID: `node-app-key` (used in the pipeline)

![](/portfolio-app-img/credentials-1.png)

![](/portfolio-app-img/credentials-2.png)

---

## Jenkins Pipeline (Jenkinsfile)

```groovy
pipeline {
    agent any

    environment {
        SSH_CRED = 'credential-id'               // Jenkins credential ID
        SERVER_IP = '0.0.0.0'               // Apache server public IP
        REMOTE_USER = 'ubuntu'
        WEB_DIR = '/var/www/html/'
    }

    stages {
        stage('Clone Repo') {
            steps {
                git url: 'https://github.com/Amogh902/Portfolio-app.git', branch: 'main'
            }
        }

        stage('Deploy to Apache Server') {
            steps {
                sshagent(credentials: ["${SSH_CRED}"]) {
                    sh '''
                        echo "Cleaning old files on server..."
                        ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${SERVER_IP} "sudo rm -rf ${WEB_DIR}/*"

                        echo "Uploading new files to /tmp on server..."
                        scp -r * ${REMOTE_USER}@${SERVER_IP}:/tmp/

                        echo "Deploying files to Apache directory..."
                        ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${SERVER_IP} "sudo cp -r /tmp/* ${WEB_DIR}/"
                    '''
                }
            }
        }
    }
}
```

---

1. Jenkins job configuration
![](/portfolio-app-img/job-configuration.png)

2. Successful Build and Console output
![](/portfolio-app-img/build-job-success.png)

3. Browser view of deployed portfolio site
![](/portfolio-app-img/browser-output.png)

4. Files stored on app-server
![](/portfolio-app-img/server-output.png)

---


