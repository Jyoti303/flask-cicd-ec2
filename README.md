# Flask CI/CD with GitHub Actions and EC2 🚀

![CI/CD](https://img.shields.io/github/actions/workflow/status/Jyoti303/flask-cicd-ec2/deploy.yml?label=Deploy%20Status)

This repository demonstrates how to set up a **Flask app** with a **CI/CD pipeline** using **GitHub Actions** and deploy it automatically to an **AWS EC2 instance** whenever changes are pushed to the `main` branch.

---

## 📋 Table of Contents
- [Prerequisites](#prerequisites)
- [Setup](#setup)
  - [Step 1: Clone the Repository](#step-1-clone-the-repository)
  - [Step 2: Set up EC2 Instance](#step-2-set-up-ec2-instance)
  - [Step 3: Set up SSH Key for GitHub Actions](#step-3-set-up-ssh-key-for-github-actions)
  - [Step 4: Create Flask App and Test Locally](#step-4-create-flask-app-and-test-locally)
  - [Step 5: Configure GitHub Secrets](#step-5-configure-github-secrets)
  - [Step 6: Set up GitHub Actions CI/CD Workflow](#step-6-set-up-github-actions-cicd-workflow)
- [CI/CD Pipeline](#cicd-pipeline)
- [How It Works](#how-it-works)
- [Testing the Pipeline](#testing-the-pipeline)

---

## 📌 Prerequisites

Make sure you have:

- **AWS Account** and **EC2 Instance** (Ubuntu server)
- **Git** installed locally
- **GitHub Account** and a **new repository**
- **SSH Key Pair** (for GitHub Actions to connect to EC2)

---

## 🛠 Setup

### Step 1: Clone the Repository

```bash
git clone https://github.com/Jyoti303/flask-cicd-ec2.git
```

---

### Step 2: Set up EC2 Instance

1. Launch an EC2 instance:
   - Ubuntu 22.04 LTS AMI
   - t2.micro instance type (free tier)
2. Connect to EC2:
   ```bash
   ssh -i "your-key.pem" ubuntu@your-ec2-public-ip
   ```
3. Install required packages:
   ```bash
   sudo apt update
   sudo apt install python3 python3-pip git -y
   pip3 install flask
   ```

---

### Step 3: Set up SSH Key for GitHub Actions

1. Generate new SSH keys locally:
   ```bash
   mkdir -p ~/.ssh/github-actions-ec2
   ssh-keygen -t ed25519 -f ~/.ssh/github-actions-ec2/id_ed25519 -C "github-actions-ec2" -N ""
   ```
2. Copy and add public key to EC2:
   ```bash
   cat ~/.ssh/github-actions-ec2/id_ed25519.pub
   ```
   - Paste the public key into `/home/ubuntu/.ssh/authorized_keys` file on EC2.

3. Add **private key** to GitHub Secrets:
   - Name: `EC2_SSH_PRIVATE_KEY`
   - Value: (paste your private key content)

---

### Step 4: Create Flask App and Test Locally

1. Create `hello.py`:

```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello():
    return '🚀 Hello from your CI/CD EC2 pipeline!'

if __name__ == '__main__':
    app.run(debug=True, host='0.0.0.0', port=5000)
```

2. Run locally to test:
   ```bash
   python3 hello.py
   ```
3. Access via browser:  
   `http://<your-ec2-public-ip>:5000`

---

### Step 5: Configure GitHub Secrets

Add these secrets:

| Secret Name          | Value                |
| -------------------- | -------------------- |
| `EC2_SSH_PRIVATE_KEY` | (your private SSH key content) |
| `EC2_PUBLIC_IP`       | (your EC2 public IP) |

---

### Step 6: Set up GitHub Actions CI/CD Workflow

Create `.github/workflows/deploy.yml`:

```yaml
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
```

Push the code to GitHub:

```bash
git add .
git commit -m "Setup CI/CD"
git push origin main
```

---

## 🔥 CI/CD Pipeline

- Triggers automatically on **push to main branch**.
- Connects via SSH to EC2.
- Pulls latest code.
- Installs dependencies.
- Restarts Flask app on the server.

---

## ⚙️ How It Works

1. You push your code to GitHub →  
2. GitHub Actions triggers →  
3. Connects to EC2 →  
4. Pulls new code →  
5. Runs Flask app automatically.

---

## 🧪 Testing the Pipeline

- Modify any code (example: change text in `hello.py`)
- Push changes:
  ```bash
  git add .
  git commit -m "Update text"
  git push origin main
  ```
- Check the app in the browser:  
  `http://<your-ec2-public-ip>:5000`

---

# 🎉 Done! Your Flask App now has a working CI/CD pipeline to EC2 using GitHub Actions!

---

## 💡 Common Troubleshooting Tips

- **Deployment Failed on GitHub Actions?**
  - Check your GitHub Actions logs carefully under the **Actions** tab.
  - Most common issues:
    - Wrong SSH key permissions.
    - EC2 IP address changed (Elastic IP recommended).
    - Python/Flask not installed on EC2.

- **Permission Denied (publickey) Error?**
  - Double-check that your SSH private key (`EC2_SSH_PRIVATE_KEY`) is correctly added in GitHub Secrets.
  - Ensure the public key is properly added to `/home/ubuntu/.ssh/authorized_keys` on the EC2 server.

- **Flask App Not Accessible?**
  - Make sure EC2 Security Group allows **Inbound Rules** for:
    - Port `5000` (for Flask app)
    - Port `22` (for SSH)

- **App URL Still Showing Old Code After Push?**
  - Sometimes EC2 may have cached processes.
  - You can manually SSH into EC2 and run:
    ```bash
    pkill -f flask
    git pull origin main
    source venv/bin/activate
    pip install -r requirements.txt
    nohup flask run --host=0.0.0.0 --port=5000 &
    ```

- **Server Crashes After Few Hours?**
  - If using **nohup**, make sure it's running properly.
  - For production, consider setting up **Gunicorn + Nginx** later (advanced topic ✨).

---

## ✨ Future Improvements
- Set up a **custom domain** (like `www.myflaskapp.com`) instead of IP address.
- Use **Elastic IP** on AWS to prevent IP address changes.
- Set up **auto-scaling** and **load balancers** for bigger apps.
- Replace Flask's built-in server with **Gunicorn** for production.

---

