# Flask EC2 CI/CD Project

This project demonstrates a CI/CD pipeline using AWS CodePipeline and CodeDeploy to deploy a Flask web app to an EC2 instance.

Flask CI/CD with GitHub Actions and EC2
This repository demonstrates how to set up a Flask app with a CI/CD pipeline using GitHub Actions and deploy it to an AWS EC2 instance. Each time changes are pushed to the main branch, the app is automatically deployed to the EC2 instance.

Table of Contents
Prerequisites

Setup

Step 1: Clone the Repository

Step 2: Set up EC2 Instance

Step 3: Set up SSH Key for GitHub Actions

Step 4: Create Flask App and Test Locally

Step 5: Configure GitHub Secrets

Step 6: Set up GitHub Actions CI/CD Workflow

CI/CD Pipeline

How It Works

Testing the Pipeline

Prerequisites
Before you begin, ensure you have the following tools installed:

AWS Account: Set up AWS EC2 instance for hosting the Flask app.

Git: Installed and configured on your machine.

SSH Key: Set up SSH keys for secure access to your EC2 instance.

GitHub Account: Repository created for hosting the Flask app.

Setup
Step 1: Clone the Repository
Clone the repository from GitHub to your local machine:

bash
Copy
Edit
git clone https://github.com/Jyoti303/flask-cicd-ec2.git
Step 2: Set up EC2 Instance
Launch an EC2 Instance:

Choose the Ubuntu 22.04 LTS AMI.

Select the t2.micro instance type (free tier eligible).

Create or use an existing SSH key pair to access the instance.

Connect to EC2 Instance:

Use SSH to connect to your EC2 instance:

bash
Copy
Edit
ssh -i "your-key.pem" ubuntu@your-ec2-public-ip
Install Required Packages: On your EC2 instance, install Python, Flask, Git, and other dependencies:

bash
Copy
Edit
sudo apt update
sudo apt install python3 python3-pip git -y
pip3 install flask
Step 3: Set up SSH Key for GitHub Actions
Generate a new SSH Key on your local machine:

bash
Copy
Edit
mkdir -p ~/.ssh/github-actions-ec2
ssh-keygen -t ed25519 -f ~/.ssh/github-actions-ec2/id_ed25519 -C "github-actions-ec2" -N ""
Copy the public key:

bash
Copy
Edit
cat ~/.ssh/github-actions-ec2/id_ed25519.pub
Add the public key to the EC2 instance’s authorized_keys:

bash
Copy
Edit
nano ~/.ssh/authorized_keys
# Paste the public key here
Add the private key to GitHub Secrets:

Go to your GitHub repository > Settings > Secrets > New repository secret.

Name: EC2_SSH_PRIVATE_KEY

Value: Paste the contents of your private key (~/.ssh/github-actions-ec2/id_ed25519).

Step 4: Create Flask App and Test Locally
Create the Flask App (e.g., hello.py):

python
Copy
Edit
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello():
    return 'Hello, World from Flask on EC2!'

if __name__ == '__main__':
    app.run(debug=True, host='0.0.0.0', port=5000)
Test locally by running:

bash
Copy
Edit
python3 hello.py
Access the app in a browser at http://<your-ec2-public-ip>:5000.

Step 5: Configure GitHub Secrets
Add EC2 Public IP to GitHub Secrets:

Go to your repository > Settings > Secrets > New repository secret.

Name: EC2_PUBLIC_IP

Value: Your EC2 public IP address.

Step 6: Set up GitHub Actions CI/CD Workflow
Create the .github/workflows/deploy.yml file in your GitHub repository:

yaml
Copy
Edit
name: Deploy Flask App to EC2

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout Code
      uses: actions/checkout@v4

    - name: Setup SSH key
      run: |
        mkdir -p ~/.ssh
        echo "${{ secrets.EC2_SSH_PRIVATE_KEY }}" > ~/.ssh/id_ed25519
        chmod 600 ~/.ssh/id_ed25519

    - name: SSH into EC2 and Deploy
      run: |
        ssh -o StrictHostKeyChecking=no -i ~/.ssh/id_ed25519 ubuntu@${{ secrets.EC2_PUBLIC_IP }} << 'EOF'
          cd /home/ubuntu/apps/flask-cicd-app
          git pull origin main
          source venv/bin/activate
          pip install -r requirements.txt
          pkill -f flask || true
          nohup flask run --host=0.0.0.0 --port=5000 &
        EOF
Push your code to GitHub:

bash
Copy
Edit
git add .
git commit -m "Set up CI/CD pipeline"
git push origin main
CI/CD Pipeline
The CI/CD pipeline is configured to:

Trigger on every push to the main branch.

Set up SSH for secure communication with the EC2 instance.

Deploy the Flask app by pulling the latest changes from GitHub, installing dependencies, and restarting the Flask app.

How It Works
GitHub Actions listens for changes on the main branch.

Once a change is detected, it:

Sets up an SSH connection to the EC2 instance using the private key stored in GitHub secrets.

Pulls the latest code from the main branch.

Installs dependencies listed in requirements.txt.

Restarts the Flask app using the flask run command.

Testing the Pipeline
Make changes to your Flask app locally.

Commit and push those changes to the main branch.

GitHub Actions will trigger the pipeline and deploy the updated code to your EC2 instance.

Verify the changes by visiting your EC2 instance’s public IP at http://<your-ec2-public-ip>:5000.

