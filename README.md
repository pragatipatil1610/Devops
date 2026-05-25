# 🏠 SmartHostel — Student Hostel Management Website

A responsive static website for student hostel management.
Stack: HTML + CSS + JS → Docker (nginx) → Jenkins CI/CD → AWS EC2

---

## 📁 Project Structure

```
DevopsHostelapp/
├── index.html
├── Dockerfile
├── docker-compose.yml
├── Jenkinsfile
├── css/style.css
├── js/main.js
├── pages/
│   ├── facilities.html
│   ├── booking.html
│   ├── dashboard.html
│   ├── complaints.html
│   └── contact.html
└── assets/
```

---

## 🔄 CI/CD Pipeline

```
GitHub Push → Webhook → Jenkins → Docker Build → DockerHub Push → SSH Deploy → AWS EC2
```

---

## ⚙️ Step-by-Step Setup

### Step 1 — AWS EC2 Setup

1. Launch an **Ubuntu 22.04** EC2 instance (t2.micro is fine)
2. Open inbound ports: **22** (SSH), **80** (HTTP), **8080** (Jenkins)
3. SSH into the instance and install Docker:

```bash
sudo apt update && sudo apt install -y docker.io
sudo systemctl enable docker && sudo systemctl start docker
sudo usermod -aG docker ubuntu
```

### Step 2 — Jenkins Setup (on EC2 or separate server)

```bash
# Install Java
sudo apt install -y openjdk-17-jdk

# Install Jenkins
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update && sudo apt install -y jenkins
sudo systemctl start jenkins

# Add jenkins user to docker group
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
```

Access Jenkins at: `http://<your-ec2-ip>:8080`

### Step 3 — Jenkins Credentials

Go to **Manage Jenkins → Credentials → Global** and add:

| ID | Type | Value |
|----|------|-------|
| `dockerhub-creds` | Username/Password | Your DockerHub login |
| `dockerhub-username` | Secret text | Your DockerHub username |
| `ec2-ssh-key` | SSH Username with private key | Your EC2 `.pem` key content |
| `ec2-host` | Secret text | Your EC2 public IP |

### Step 4 — Jenkins Pipeline Job

1. New Item → **Pipeline**
2. Under **Pipeline**, select **Pipeline script from SCM**
3. SCM: **Git**, URL: `https://github.com/28092005/Devops-lab.git`
4. Branch: `*/main`
5. Script Path: `Jenkinsfile`
6. Save

### Step 5 — GitHub Webhook

1. Go to your repo → **Settings → Webhooks → Add webhook**
2. Payload URL: `http://<jenkins-ip>:8080/github-webhook/`
3. Content type: `application/json`
4. Trigger: **Just the push event**
5. Save

In Jenkins job → **Build Triggers** → check **GitHub hook trigger for GITScm polling**

### Step 6 — Update docker-compose.yml

Replace `YOUR_DOCKERHUB_USERNAME` in `docker-compose.yml` with your actual DockerHub username.

---

## 🚀 How It Works After Setup

1. You push code to `main` branch on GitHub
2. GitHub sends webhook to Jenkins
3. Jenkins pulls code, builds Docker image, tags with build number
4. Image pushed to DockerHub
5. Jenkins SSHs into EC2, pulls latest image, restarts container
6. Site is live at `http://<ec2-ip>`

---

## 🌿 Branching Strategy

```
main      → triggers production deploy
staging   → for testing (add another pipeline stage if needed)
feature/* → no auto-deploy
```

---

## 📦 Version History

| Version | Changes |
|---------|---------|
| v1.0.0  | Initial release — all 6 pages |
| v1.1.0  | Added Docker + Jenkins + AWS EC2 pipeline |

---

*SmartHostel © 2025 — Built for DevOps CI/CD demonstration*
