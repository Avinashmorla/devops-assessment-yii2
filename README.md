# 🚀 End-to-End CI/CD Pipeline for Yii2 using Docker, Ansible & GitHub Action

## 📌 Objective

This project demonstrates the deployment of a PHP-based Yii2 application using:

- **Docker Swarm** for container orchestration  
- **NGINX** as a reverse proxy  
- **Ansible** for infrastructure automation  
- **GitHub Actions** for CI/CD

The entire stack runs on an **AWS EC2 instance**, making it a real-world DevOps deployment scenario.


## 🗂️ Project Structure

```bash
devops-assignment/
├── ansible/                   
│   ├── configure_nginx.yml    
│   ├── deploy_app.yml         
│   ├── init_swarm.yml         
│   ├── inventory              
│   ├── mykeypair.pem          
│   ├── nginx_app.conf        
│   └── setup_instance.yml     
├── yii2-app/                  
│   ├── Dockerfile             
│   ├── composer.json          
│   ├── docker-compose.yml     
│   └── web/
│       └── index.php          
└── .github/
    └── workflows/
        └── deploy.yml         
```


## ⚙️ Prerequisites

- AWS account with access to launch EC2 instances
- EC2 key pair (`mykeypair.pem`)
- Docker Hub account
- GitHub account with a **private** repository
- Local machine or Google Cloud Shell with:
  - Ansible
  - Git
  - SSH client


## ☁️ 1. EC2 Instance Setup (Manual - Initial Only)

1. **Launch EC2:**
   - Ubuntu Server 22.04 LTS
   - Open ports: `22` (SSH), `80` (HTTP)
   - Associate your `mykeypair.pem`

2. **Prepare SSH Key:**
   - Copy `mykeypair.pem` into `ansible/` folder
   - Set permissions:
     ```bash
     chmod 400 mykeypair.pem
     ```

3. **Update Inventory File:**
   - Edit `ansible/inventory` with your EC2 Public I

## 🔧 2. Infrastructure Setup with Ansible

Run the following from the `ansible/` directory:

```bash
cd ~/devops-assignment/ansible

# Install Docker, Git, and dependencies
ansible-playbook -i inventory setup_instance.yml

# Initialize Docker Swarm
ansible-playbook -i inventory init_swarm.yml

# Set up NGINX reverse proxy
ansible-playbook -i inventory configure_nginx.yml

# Deploy Yii2 app as a Docker Swarm stack
ansible-playbook -i inventory deploy_app.yml
```


## 🔗 3. GitHub Repository Setup

```bash
cd ~/devops-assignment
git init
git add .
git commit -m "Initial commit"

# Create a private repo on GitHub (without README/init files)
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/devops-assessment-yii2.git
git push -u origin main
```


## 🔐 4. Configure GitHub Secrets (CI/CD)

Go to:  
`GitHub > Repo > Settings > Secrets and Variables > Actions`

| Secret Name         | Description |
|---------------------|-------------|
| `SSH_PRIVATE_KEY`   | Private deploy key to access EC2 |
| `DOCKER_HUB_USERNAME` | Your Docker Hub username |
| `DOCKER_HUB_TOKEN`    | Docker Hub personal access token |

### Generate SSH Key for CI/CD:

```bash
ssh-keygen -t rsa -b 4096 -C "github-actions-deploy-key" -f ~/.ssh/github_actions_deploy_key -N ""
```

- Add **private key** (`cat ~/.ssh/github_actions_deploy_key`) to GitHub Secrets as `SSH_PRIVATE_KEY`
- Add **public key** (`cat ~/.ssh/github_actions_deploy_key.pub`) to EC2:
  ```bash
  ssh -i mykeypair.pem ubuntu@YOUR_EC2_IP
  echo "PASTE_PUBLIC_KEY" >> ~/.ssh/authorized_keys
  ```

## ⚙️ 5. Set Up GitHub Actions Workflow

1. Create workflow file:
   ```bash
   mkdir -p .github/workflows
   touch .github/workflows/deploy.yml
   ```

2. Paste your workflow YAML (`deploy.yml`) into this file.

3. Commit and push:
   ```bash
   git add .github/workflows/deploy.yml
   git commit -m "Add CI/CD workflow"
   git push origin main
   ```

This triggers the first pipeline run automatically.

## 🧪 How to Test

### ✅ Initial Manual Deployment
- Visit: `http://<your-ec2-ip>`  
- You should see:  
  **"Hello from Yii2 Docker Swarm!"**

### 🔁 Test CI/CD Workflow
1. Modify `yii2-app/web/index.php`:
   ```php
   echo "Hello from CI/CD Pipeline!";
   ```

2. Commit and push:
   ```bash
   git add yii2-app/web/index.php
   git commit -m "Update index.php for CI/CD test"
   git push origin main
   ```

3. Check **Actions** tab on GitHub for workflow execution.

4. Refresh browser: `http://<your-ec2-ip>`  
   Output should reflect the updated message.



## 🧾 Assumptions

- EC2 user is `ubuntu` with passwordless sudo
- Port 80 is open for HTTP access
- App is stateless and doesn't require persistent DB
- Docker image: `your-dockerhub-username/yii2-app:latest`
