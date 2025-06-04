# DevOps Assessment - Yii2 + Docker Swarm + CI/CD + Ansible

## Objective
This project demonstrates the deployment of a sample PHP Yii2 application using Docker Swarm and NGINX as a host-based reverse proxy on an AWS EC2 instance. The entire setup is automated using Ansible, and a CI/CD pipeline is implemented via GitHub Actions for continuous deployment.

## Project Structure
devops-assignment/
├── ansible/
│   ├── configure_nginx.yml
│   ├── deploy_app.yml
│   ├── init_swarm.yml
│   ├── inventory
│   ├── mykeypair.pem (EC2 SSH key - exclude from public repo if sensitive)
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

## Setup Instructions

### Prerequisites
* An AWS account with permissions to launch EC2 instances and create key pairs.
* AWS EC2 Key Pair (`mykeypair.pem`) generated and accessible.
* Docker Hub account for storing Docker images.
* GitHub account and a **private** repository for this project.
* Google Cloud Shell (or a local machine with Ansible, Git, SSH client installed).

### 1. EC2 Instance Provisioning (Manual First Time)
1.  **Launch an EC2 Instance:**
    * Launch an Ubuntu Server 22.04 LTS (HVM), SSD Volume Type.
    * Ensure security group allows SSH (port 22) from your IP and HTTP (port 80) from anywhere.
    * Associate your `mykeypair.pem` with the instance.
    * Note down the Public IP address of the instance.
2.  **Place SSH Key:** Copy your `mykeypair.pem` file into the `ansible/` directory. Ensure its permissions are `chmod 400 mykeypair.pem`.
3.  **Update Ansible Inventory:** In `ansible/inventory`, replace `your_ec2_ip_address` with your actual EC2 Public IP.

### 2. Initial Infrastructure Setup with Ansible
From your Cloud Shell (or local machine):
1.  Navigate to the `ansible` directory:
    ```bash
    cd ~/devops-assignment/ansible
    ```
2.  Run the setup playbook to install Docker, NGINX, Git, and other dependencies:
    ```bash
    ansible-playbook -i inventory setup_instance.yml
    ```
3.  Initialize Docker Swarm:
    ```bash
    ansible-playbook -i inventory init_swarm.yml
    ```
4.  Configure NGINX:
    ```bash
    ansible-playbook -i inventory configure_nginx.yml
    ```
5.  Deploy the Yii2 application as a Docker Swarm service for the first time:
    ```bash
    ansible-playbook -i inventory deploy_app.yml
    ```

### 3. GitHub Repository Setup
1.  **Initialize Git:**
    ```bash
    cd ~/devops-assignment
    git init
    git add .
    git commit -m "Initial commit of DevOps assessment project"
    ```
2.  **Create Private GitHub Repo:** Go to `https://github.com/new`, create a **private** repository (e.g., `devops-assessment-yii2`), and **do NOT** initialize it with a README or any other files.
3.  **Link and Push:**
    ```bash
    git branch -M main
    git remote add origin [https://github.com/YOUR_GITHUB_USERNAME/devops-assessment-yii2.git](https://github.com/YOUR_GITHUB_USERNAME/devops-assessment-yii2.git)
    git push -u origin main
    ```
    (You'll need a GitHub Personal Access Token with `repo` and `workflow` scopes instead of your password for `git push`).

### 4. Configure GitHub Secrets for CI/CD
In your GitHub repository (`Settings` > `Secrets and variables` > `Actions`):
1.  **`SSH_PRIVATE_KEY`**:
    * Generate a new SSH key pair on Cloud Shell:
        ```bash
        ssh-keygen -t rsa -b 4096 -C "github-actions-deploy-key" -f ~/.ssh/github_actions_deploy_key -N ""
        ```
    * Copy the **entire private key content** (`cat ~/.ssh/github_actions_deploy_key`) and paste it into this secret.
    * Add the **public key** (`cat ~/.ssh/github_actions_deploy_key.pub`) to your EC2 instance's `~/.ssh/authorized_keys` file:
        ```bash
        ssh -i ~/.ssh/mykeypair.pem ubuntu@YOUR_EC2_IP
        echo "PASTE_PUBLIC_KEY_HERE" >> ~/.ssh/authorized_keys
        exit
        ```
2.  **`DOCKER_HUB_USERNAME`**: Your Docker Hub username.
3.  **`DOCKER_HUB_TOKEN`**: Your Docker Hub Personal Access Token (recommended) with "Read & Write" permissions.

### 5. GitHub Actions CI/CD Workflow
1.  Create the workflow directory and file:
    ```bash
    mkdir -p .github/workflows
    vi .github/workflows/deploy.yml
    ```
2.  Paste the content provided in our discussion (the `deploy.yml` content) into this file.
3.  Commit and push the workflow file:
    ```bash
    git add .github/workflows/deploy.yml
    git commit -m "Add GitHub Actions CI/CD workflow"
    git push origin main
    ```
    This push will trigger your first CI/CD pipeline run.

## Assumptions
* The EC2 instance is running Ubuntu Server 22.04 LTS.
* The `ubuntu` user has `sudo` privileges without requiring a password (common in EC2).
* Port 80 (HTTP) is open on the EC2 security group for web access.
* Your `mykeypair.pem` is correctly configured to SSH into the EC2 instance.
* The Yii2 application is basic and does not require complex database migrations or persistent storage beyond the scope of this deployment.
* Docker Hub image naming convention: `YOUR_DOCKER_HUB_USERNAME/yii2-app:latest`.

## How to Test Deployment

### Initial Verification
1.  After the initial Ansible deployment (`deploy_app.yml`), open your browser and navigate to your EC2 instance's Public IP address (e.g., `http://13.234.75.81`). You should see "Hello from Yii2 Docker Swarm!"

### Testing CI/CD Pipeline
1.  Make a small, visible change to the `yii2-app/web/index.php` file (e.g., change the text to "Hello from CI/CD Pipeline!").
2.  Commit the change:
    ```bash
    git add yii2-app/web/index.php
    git commit -m "Update index.php for CI/CD test"
    ```
3.  Push the changes to your `main` branch:
    ```bash
    git push origin main
    ```
4.  Go to your GitHub repository's `Actions` tab. You should see a new workflow run initiated.
5.  Wait for the workflow to complete successfully (green checkmark).
6.  Refresh your browser at your EC2 instance's Public IP (`http://13.234.75.81`).
