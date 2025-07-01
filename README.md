# 🚀 AWS CodeDeploy Pipeline Demo

> A complete guide to setting up automated deployments with AWS CodeDeploy and CodePipeline

## 📋 Table of Contents

- [🎯 Overview](#-overview)
- [📁 Project Structure](#-project-structure)
- [🔧 Setup Guide](#-setup-guide)
- [🔑 IAM Roles Configuration](#-iam-roles-configuration)
- [🖥️ EC2 Instance Setup](#️-ec2-instance-setup)
- [⚙️ CodeDeploy Configuration](#️-codedeploy-configuration)
- [🔄 CodePipeline Setup](#-codepipeline-setup)
- [📄 File Explanations](#-file-explanations)
- [🚨 Troubleshooting](#-troubleshooting)

## 🎯 Overview

This project demonstrates how to create a fully automated CI/CD pipeline using AWS CodeDeploy and CodePipeline to deploy a simple web application to EC2 instances. Perfect for learning AWS DevOps fundamentals!

**What you'll build:**

- 🏗️ Automated deployment pipeline
- 🖥️ EC2 web server with Apache
- 📦 Code deployment from GitHub
- 🔄 Continuous integration workflow

## 📁 Project Structure

```
aws-pipeline-demo/
├── 📄 appspec.yml           # CodeDeploy application specification
├── 🌐 index.html           # Main web page
├── 🔧 user-data.sh         # EC2 initialization script
├── 📂 images/              # Website images and assets
├── 📂 lib/                 # External libraries (Montserrat fonts)
├── 📂 styles/              # CSS stylesheets
└── 📂 scripts/             # Deployment lifecycle scripts
    ├── install-dependencies.sh
    ├── start-server.sh
    └── stop-server.sh
```

## 🔧 Setup Guide

Follow these steps in order to create your automated deployment pipeline:

## 🔑 IAM Roles Configuration

### Step 1A: 🤖 EC2 CodeDeploy Role

This role allows EC2 instances to receive and execute deployments from CodeDeploy.

1. 🔍 Navigate to **IAM Console** → **Roles** → **Create Role**
2. ⚙️ **Trusted Entity Type**: AWS Service
3. 🎯 **Use Case**: EC2
4. 🔒 **Permissions**: Search for "codedeploy" and select:
   - `AmazonEC2RoleForAWSCodeDeploy`
5. 📝 **Role Name**: `EC2CodeDeploy`
6. ✅ Click **Create Role**

### Step 1B: 🚀 CodeDeploy Service Role

This role allows CodeDeploy to manage deployments on your behalf.

1. 🔍 Navigate to **IAM Console** → **Roles** → **Create Role**
2. ⚙️ **Trusted Entity Type**: AWS Service
3. 🎯 **Use Case**: CodeDeploy
4. 🔒 **Permissions**: The role `AWSCodeDeployRole` is automatically selected
5. 📝 **Role Name**: `CodeDeployRole`
6. ✅ Click **Create Role**

## 🖥️ EC2 Instance Setup

### Step 2: 🌐 Launch EC2 Instance

Create an EC2 instance that will host your web application.

1. 🔍 Navigate to **EC2 Console** → **Launch Instance**
2. 📝 **Name**: `cdServer` (CodeDeploy Server)
3. 🐧 **AMI**: Amazon Linux 2023 (or Amazon Linux 2)
4. 💻 **Instance Type**: t2.micro (free tier eligible)
5. 🔑 **Key Pair**: Create or select existing key pair for SSH access

#### 🛡️ Security Group Configuration:

- **SSH (22)**: Your IP (for troubleshooting)
- **HTTP (80)**: 0.0.0.0/0 (public web access)

#### ⚙️ Advanced Details:

- **IAM Instance Profile**: `EC2CodeDeploy` (the role we created)
- **User Data**: Copy the content from `user-data.sh` in this repository

#### 🏷️ Tags:

- **Key**: `Name`
- **Value**: `cdServer`

> ⚠️ **Important**: The tag `Name: cdServer` is crucial for CodeDeploy to identify this instance!

## ⚙️ CodeDeploy Configuration

### Step 3: 📦 Create CodeDeploy Application

1. 🔍 Navigate to **CodeDeploy Console** → **Applications** → **Create Application**
2. 📝 **Application Name**: `codeDeployDemo`
3. 🖥️ **Compute Platform**: EC2/On-premises
4. ✅ Click **Create Application**

### Step 3.1: 👥 Create Deployment Group

1. 📂 In your application, click **Create Deployment Group**
2. 📝 **Deployment Group Name**: `demo-dg-group`
3. 🔑 **Service Role**: `CodeDeployRole` (the role we created)
4. 🎯 **Environment Configuration**:
   - ✅ Check **Amazon EC2 instances**
   - **Tag**: `Name` = `cdServer`
5. ⚙️ **Deployment Settings**: `CodeDeployDefault.AllAtOnce`
6. 🚫 **Load Balancer**: Uncheck (we're using a single instance)
7. ✅ Click **Create Deployment Group**

## 🔄 CodePipeline Setup

### Step 4: 🔄 Create Pipeline

1. 🔍 Navigate to **CodePipeline Console** → **Create Pipeline**
2. 📝 **Pipeline Name**: `codeDeployPipeline`
3. ⚙️ **Service Role**: Create new service role

#### 📥 Source Stage:

1. **Source Provider**: GitHub (Version 2) via GitHub App
2. 🔗 **Connection**: Create new connection or select existing
3. 📁 **Repository**: Select your repository from dropdown
4. 🌿 **Branch**: `main`
5. 📤 **Output Artifacts**: `SourceOutput`

#### 🏗️ Build Stage:

- ⏭️ **Skip Build Stage** (we're deploying static files)

#### 🧪 Test Stage:

- ⏭️ **Skip Test Stage**

#### 🚀 Deploy Stage:

1. **Deploy Provider**: AWS CodeDeploy
2. 📦 **Application Name**: `codeDeployDemo`
3. 👥 **Deployment Group**: `demo-dg-group`
4. 📥 **Input Artifacts**: `SourceOutput`

## 📄 File Explanations

### 📋 `appspec.yml` - Deployment Specification

```yaml
version: 0.0
os: linux
files:
  - source: /index.html
    destination: /var/www/html/
  - source: /images/
    destination: /var/www/html/images/
  - source: /lib/
    destination: /var/www/html/lib/
  - source: /styles/
    destination: /var/www/html/styles/
hooks:
  BeforeInstall:
    - location: scripts/install-dependencies.sh
      timeout: 300
      runas: root
    - location: scripts/start-server.sh
      timeout: 300
      runas: root
  ApplicationStop:
    - location: scripts/stop-server.sh
      timeout: 300
      runas: root
```

**What it does:**

- 📁 **Files Section**: Copies website files to Apache web directory
- 🔄 **Hooks Section**: Defines scripts to run during deployment lifecycle
  - `BeforeInstall`: Installs dependencies and starts Apache
  - `ApplicationStop`: Stops Apache before new deployment

### 🔧 `user-data.sh` - EC2 Initialization Script

This script runs when the EC2 instance first boots up:

```bash
#!/bin/bash
sudo yum -y update                    # Update system packages
sudo yum -y install ruby             # Install Ruby (required for CodeDeploy agent)
sudo yum -y install wget             # Install wget for downloading files
cd /home/ec2-user                     # Navigate to user home directory
# Download CodeDeploy agent installer for ap-south-1 region
wget https://aws-codedeploy-ap-south-1.s3.ap-south-1.amazonaws.com/latest/install
sudo chmod +x ./install              # Make installer executable
sudo ./install auto                  # Install CodeDeploy agent automatically
sudo yum install -y python-pip       # Install Python package manager
sudo pip install awscli              # Install AWS CLI (optional)
```

### 📂 `scripts/` Directory

#### 🔧 `install-dependencies.sh`

Installs and configures Apache web server:

```bash
#!/bin/bash
sudo yum update -y
sudo yum install -y httpd
sudo systemctl enable httpd
```

#### ▶️ `start-server.sh`

Starts the Apache web server:

```bash
#!/bin/bash
sudo systemctl start httpd
sudo systemctl status httpd
```

#### ⏹️ `stop-server.sh`

Stops the Apache web server:

```bash
#!/bin/bash
sudo systemctl stop httpd
```

### 🌐 `index.html` - Main Web Page

Your beautiful static website that will be deployed automatically!

### 🎨 `styles/main.css` - Stylesheet

Contains all the CSS styling for your website.

### 🖼️ `images/` - Assets Directory

Contains logos, background images, and other visual assets.

### 📚 `lib/` - Libraries Directory

Contains external libraries like the Montserrat font family.

## 🚨 Troubleshooting

### Common Issues and Solutions:

#### 🔴 Deployment Fails - "No instances found"

- ✅ Check EC2 instance has correct tag: `Name: cdServer`
- ✅ Verify instance is running and healthy
- ✅ Ensure IAM role `EC2CodeDeploy` is attached to instance

#### 🔴 CodeDeploy Agent Not Running

- 🔧 SSH into instance: `sudo service codedeploy-agent status`
- 🔄 Restart agent: `sudo service codedeploy-agent restart`
- 📋 Check logs: `sudo tail -f /var/log/aws/codedeploy-agent/codedeploy-agent.log`

#### 🔴 Scripts Fail During Deployment

- 📋 Check script permissions are executable
- 🔍 Verify script paths in `appspec.yml`
- 📝 Check deployment logs in CodeDeploy console

#### 🔴 Website Not Accessible

- 🛡️ Verify security group allows HTTP (port 80)
- 🔧 Check Apache status: `sudo systemctl status httpd`
- 📁 Verify files are in `/var/www/html/`

#### 🔴 Pipeline Source Connection Issues

- 🔗 Recreate GitHub connection in CodePipeline
- ✅ Verify repository permissions
- 🌿 Check branch name is correct

### 📋 Useful Commands:

```bash
# Check CodeDeploy agent status
sudo service codedeploy-agent status

# View deployment logs
sudo tail -f /var/log/aws/codedeploy-agent/codedeploy-agent.log

# Check Apache status
sudo systemctl status httpd

# View Apache error logs
sudo tail -f /var/log/httpd/error_log

# List files in web directory
ls -la /var/www/html/
```

## 🎉 Success!

Once everything is set up correctly:

1. 📝 **Make changes** to your code and push to GitHub
2. 🔄 **Pipeline automatically triggers** and deploys changes
3. 🌐 **Visit your EC2 public IP** to see the updated website
4. 🎊 **Enjoy your automated deployment pipeline!**

---

**Happy Deploying!** 🚀

> 💡 **Pro Tip**: Monitor your deployments in the CodeDeploy console and check the deployment history to track your releases!
