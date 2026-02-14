🚀 SmartDeploy – Automated CI/CD Pipeline

SmartDeploy is an end-to-end **CI/CD pipeline** built using Jenkins, SonarQube, Maven, and Apache Tomcat to automate build, testing, code quality analysis, and deployment of a Java-based Ecommerce web application.

This project demonstrates a complete **DevOps workflow** with automated integration, code quality checks, deployment, and email notifications.

---

## 📌 Project Overview

This pipeline automates the following processes:

✅ Source code retrieval from GitHub  
✅ Build and compile using Maven  
✅ Automated testing  
✅ Static code analysis using SonarQube  
✅ Code quality report generation  
✅ Deployment to Apache Tomcat server  
✅ Email notifications on build status  
✅ Automated report push to GitHub  

---

## 🏗 Architecture Workflow



GitHub Repository
↓
Jenkins Pipeline
↓
Build → Test → SonarQube Analysis
↓
Package Application (.war)
↓
Deploy to Apache Tomcat
↓
Email Notification


---

## ⚙️ Tech Stack

- **CI/CD Tool:** Jenkins
- **Code Quality Tool:** SonarQube
- **Build Tool:** Maven
- **Application Server:** Apache Tomcat
- **Database:** PostgreSQL (SonarQube)
- **Cloud Platform:** AWS EC2
- **Programming Language:** Java
- **Version Control:** Git & GitHub

---

## 🧩 Features

- Fully automated CI/CD pipeline
- Static code analysis with quality gates
- Continuous integration with GitHub
- Automatic deployment to Tomcat
- Email notification system
- SonarQube report generation
- Secure credential handling
- Production-ready pipeline structure

---

## 🚀 Pipeline Stages

1. Clone Repository
2. Check Project Structure
3. Compile Application
4. Run Unit Tests
5. SonarQube Code Analysis
6. Download Sonar Report
7. Push Report to GitHub
8. Package Application
9. Deploy to Tomcat Server
10. Send Email Notification

---

## 🔧 Installation & Setup

### 1️⃣ Launch AWS EC2 Instances
- Jenkins Server
- SonarQube Server

Open required ports:
- 8080 → Jenkins
- 9000 → SonarQube
- 9090 → Tomcat

---

### 2️⃣ Install Jenkins

sudo apt update
sudo apt install openjdk-21-jre -y
sudo apt install maven -y
sudo apt install jenkins -y


Start Jenkins:

sudo systemctl start jenkins

3️⃣ Install SonarQube

Install PostgreSQL

Configure SonarQube database

Start SonarQube service

Access:

http://<server-ip>:9000


Generate authentication token.

4️⃣ Configure Jenkins

Install Plugins:

SonarQube Scanner

Maven Integration

Email Extension Plugin

Configure:

Manage Jenkins → Tools → Maven
Manage Jenkins → System → SonarQube Server
Manage Jenkins → Credentials → Add Sonar Token

5️⃣ Setup Apache Tomcat
wget https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.115/bin/apache-tomcat-9.0.115.zip
unzip apache-tomcat-9.0.115.zip
cd apache-tomcat-9.0.115/bin
./startup.sh


Access:

http://<server-ip>:9090

📬 Email Notification Setup

Configure Gmail SMTP in Jenkins:

SMTP Server: smtp.gmail.com
Port: 465
Use SSL: Yes
Use Google App Password

📊 SonarQube Quality Gate

Quality conditions include:

Coverage < 70%

Duplicated lines > 3%

Block severity issues > 0

Pipeline fails if quality gate fails.

🎯 Learning Outcomes

CI/CD pipeline implementation

Jenkins pipeline scripting

SonarQube integration

Automated deployment

Cloud infrastructure setup

DevOps best practices

👨‍💻 Author

Suman M
Artificial Intelligence & Data Science Graduate
DevOps | Machine Learning | Cloud Enthusiast

⭐ Future Improvements

Docker container deployment

Kubernetes integration

Nexus artifact repository

Blue-green deployment

Monitoring with Prometheus & Grafana

📜 License

This project is for educational and demonstration purposes.
