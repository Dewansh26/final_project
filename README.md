Here's your `README.md` file:

---

# 🚀 Chatbot Deployment with CI/CD

## 📖 Overview
This project automates the **CI/CD pipeline** for deploying a chatbot application using:
- **Jenkins** for automation
- **Terraform** for infrastructure provisioning
- **Docker** for containerization
- **Trivy & OWASP Dependency Check** for security scanning
- **SonarQube** for code quality analysis
- **AWS & Kubernetes** for cloud deployment

---

## 🛠️ Technologies Used
- **Jenkins**: CI/CD automation
- **Terraform**: Infrastructure as Code (IaC)
- **Docker**: Containerization
- **Kubernetes**: Container orchestration
- **AWS**: Cloud deployment
- **SonarQube**: Code quality analysis
- **Trivy**: Security scanning
- **OWASP Dependency Check**: Vulnerability scanning
- **Node.js**: Application runtime

---

## 📌 CI/CD Pipeline Stages

### 1️⃣ **Checkout from GitHub**
```groovy
stage('Checkout from Git') {
    steps {
        git branch: 'master', url: 'https://github.com/Dewansh26/final_project.git'
    }
}
```

### 2️⃣ **Install Dependencies**
```groovy
stage('Install Dependencies') {
    steps {
        sh "npm install"
    }
}
```

### 3️⃣ **SonarQube Code Analysis**
```groovy
stage("Sonarqube Analysis") {
    steps {
        withSonarQubeEnv('sonar-server') {
            sh '''
            $SCANNER_HOME/bin/sonar-scanner \
            -Dsonar.projectName=Chatbot \
            -Dsonar.projectKey=Chatbot
            '''
        }
    }
}
```

### 4️⃣ **Quality Gate Check**
```groovy
stage("Quality Gate") {
    steps {
        script {
            waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
        }
    }
}
```

### 5️⃣ **Security Scanning (OWASP & Trivy)**
```groovy
stage('OWASP FS SCAN') {
    steps {
        dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
        dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
    }
}

stage('TRIVY FS SCAN') {
    steps {
        sh "trivy fs . > trivyfs.json"
    }
}
```

### 6️⃣ **Docker Build & Push**
```groovy
stage("Docker Build & Push") {
    steps {
        script {
            withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                sh "docker build -t chatbot ."
                sh "docker tag chatbot dewansh26/chatbot:latest"
                sh "docker push dewansh26/chatbot:latest"
            }
        }
    }
}
```

### 7️⃣ **Container Security Scan with Trivy**
```groovy
stage("TRIVY Image Scan") {
    steps {
        sh "trivy image dewansh26/chatbot:latest > trivy.json"
    }
}
```

### 8️⃣ **Remove Old Container**
```groovy
stage("Remove Old Container") {
    steps {
        sh "docker stop chatbot || true"
        sh "docker rm chatbot || true"
    }
}
```

### 9️⃣ **Deploy to Docker Container**
```groovy
stage('Deploy to Container') {
    steps {
        sh 'docker run -d --name chatbot -p 3000:3000 dewansh26/chatbot:latest'
    }
}
```

---

## ⚙️ Terraform Setup

### **Install Terraform**
```bash
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform -y
```

### **Terraform Commands**
```bash
terraform init
terraform validate
terraform plan
terraform apply --auto-approve
```

---

## 🔍 Troubleshooting

### **Jenkins Not Starting?**
Check logs:
```bash
sudo journalctl -u jenkins --since "yesterday"
```
Fix Java issue:
```bash
java -version  # Ensure Java 17+ is installed
```
Restart Jenkins:
```bash
sudo systemctl restart jenkins
```

---

## 📌 Author
**Dewansh26** - DevOps Engineer & Cloud Enthusiast

---

This `README.md` should serve as a detailed guide to your project! 🚀 Let me know if you need modifications.