#  Node Application Deployment using CI/CD
## Project Overview
This project demonstrates deployment of a **Node.js application** using **CI/CD pipeline** with **Jenkins** on **AWS**.  
Whenever code is pushed to the GitHub repository, the pipeline automatically **builds, tests, and deploys** the application.
## Architecture
This Node.js CI/CD pipeline demonstrates automated deployment of a Node.js application on an AWS EC2 instance using Jenkins, GitHub, and PM2.

![Architecture](images/img-1.png)


## Technologies & Platforms
- **AWS EC2** – Hosting Node.js application and Jenkins  
- **GitHub** – Source code version control  
- **Jenkins** – CI/CD automation  
- **Node.js** – Application runtime  
- **PM2** – Keeps Node.js app running continuously  
- **Ubuntu** – Server OS  

## Steps:
### Step 1: Launch EC2 Instances

#### 1. Launch 2 Ubuntu EC2 instances:
    
  1. Jenkins Server – for CI/CD
  2. Node.js Application Server – to host your app

#### 2. Configure Security Groups:

   1. SSH (22)
   2. HTTP (80) / Node.js port (3000)
   3. Jenkins port (8080)
   4. Note public IPs for future access.

   ![Architecture](images/img1.png)


### Step 2️: Create & Clone GitHub Repository

#### 1. Create a new repository: Node-app-deployment-using-CICD-pipeline

#### 2. Clone repository on local machine using Git Bash:
 
1. git clone https://github.com/Ashwini2601/Node-app-deployment-using-CICD-pipeline.git
2. cd Node-app-deployment-using-CICD-pipeline

 
![Architecture](images/img2.png)

### Step 3:Create Application & Jenkins Configuration Files

 #### 1. app.js
 
```const express = require('express');
const app = express();
const port = 3000;

app.get('/', (req, res) => {
  res.send('Hello from jenkins, and pm2 deployed node.js App through jenkins cicd pipeline!');
});

app.listen(port, () => {
  console.log(`App listening at http://localhost:${port}`);
});

 ```
 #### 2. jenkinsfile
```
 pipeline {
     agent any

    environment {
        SERVER_IP      = '13.125.195.62'
        SSH_CREDENTIAL = 'node-app-key'
        REPO_URL       = 'https://github.com/Ashwini2601/Node-app-deployment-using-CICD-pipeline.git'
        BRANCH         = 'main'
        REMOTE_USER    = 'ubuntu'
        REMOTE_PATH    = '/home/ubuntu/node-app'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: "${BRANCH}", url: "${REPO_URL}"
            }
        }

        stage('Upload Files to EC2') {
            steps {
                sshagent([SSH_CREDENTIAL]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${SERVER_IP} 'mkdir -p ${REMOTE_PATH}'
                        scp -o StrictHostKeyChecking=no -r * ${REMOTE_USER}@${SERVER_IP}:${REMOTE_PATH}/
                    """
                }
            }
        }

        stage('Install Dependencies & Start App') {
            steps {
                sshagent([SSH_CREDENTIAL]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${SERVER_IP} '
                            cd ${REMOTE_PATH} &&
                            npm install &&
                            pm2 start app.js --name node-app || pm2 restart node-app
                        '
                    """
                }
            }
        }
    }

    post {
        success {
            echo '✅ Application deployed successfully!'
        }
        failure {
            echo '❌ Deployment failed.'
        }
    }
}

 ```

#### 3. package.json
```
{
  "name": "jenkins-node-app",
  "version": "1.0.0",
  "main": "app.js",
  "scripts": {
    "start": "node app.js"
  },
  "dependencies": {
    "express": "^4.18.2"
  }
}
 ```
#### 4. Commit & push:
1. git add .
2. git commit -m "Added all files to github"
3. git push -u origin main 

![Architecture](images/img3.png)

### Step 4: Install Required Plugins
1. SSH Agent
2. Git
3. Pipeline

### Step 5: Create Jenkins Credentials

#### 1. Go to Manage Jenkins → Credentials → Global → Add Credentials

#### 2. Add SSH key for EC2:

1. Kind: SSH Username with private key
2. Username: ubuntu
3. Private Key: paste key or choose file
4. ID: node-app-key

![Architecture](images/img4.png)

### Step 6: Create Jenkins Job

#### 1.New Item → Pipeline → Name: NodeJS-CICD-Pipeline
#### 2.Pipeline Definition → Pipeline script from SCM → Git
#### 3.Repository URL, branch, and credentials (node-app-key)
#### 4.Save & Build Now

### Step 7: Test Deployment

#### 1. Trigger Build Now in Jenkins

#### 2. Check Console Output

![Architecture](images/img5.png)

#### 3.Access app in browser:http://<EC2-PUBLIC-IP:3000
 

![Architecture](images/img6.png)

#### Your Node.js app is now deployed and running using Jenkins CI/CD Pipeline with PM2 on AWS EC2!
