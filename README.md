# 🚀 Jenkins-Tomcat Java Web App CI/CD Pipeline

This project demonstrates how to set up a CI/CD pipeline using Jenkins to automatically build and deploy a basic Java web application to Apache Tomcat.

---

## 🧰 Tech Stack

- Java (Servlet-based Web App)
- Apache Tomcat 9
- Jenkins
- Maven
- GitHub
- Ubuntu (or other Linux-based server)

---

## 📁 Project Structure

```
jenkins-tomcat-java-cicd/
├── src/
│   └── main/
│       ├── java/
│       │   └── HelloServlet.java
│       └── webapp/
│           └── WEB-INF/
│               └── web.xml
├── pom.xml
├── Jenkinsfile
└── README.md
```

---

## 🛠️ Step-by-Step Setup

### 1️⃣ Clone This Repo

```bash
git clone https://github.com/perumandlahemakumari/jenkins-tomcat-java-cicd.git
cd jenkins-tomcat-java-cicd
```

---

### 2️⃣ Java App Overview

**`HelloServlet.java`**:
A simple servlet that prints "Hello from Java Web App!".

**`web.xml`**:
Servlet configuration for mapping `/hello` URL to `HelloServlet`.

**`pom.xml`**:
Maven configuration to build a `.war` file.

---

### 3️⃣ Install Jenkins

```bash
sudo apt update
sudo apt install openjdk-11-jdk
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt install jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

Access Jenkins at: `http://<your-ip>:8080`

---

### 4️⃣ Install Tomcat 9

```bash
cd /opt
sudo wget https://downloads.apache.org/tomcat/tomcat-9/v9.0.85/bin/apache-tomcat-9.0.85.tar.gz
sudo tar -xvzf apache-tomcat-9.0.85.tar.gz
sudo mv apache-tomcat-9.0.85 tomcat9
```

Start Tomcat:
```bash
sudo ./tomcat9/bin/startup.sh
```

Access at: `http://<your-ip>:8081`

---

### 5️⃣ Configure Jenkins

#### Install Plugins:
- Git Plugin
- Maven Integration Plugin
- Deploy to container Plugin
- SSH Agent Plugin

#### Setup Freestyle Job:
1. Source Code: Git > this repo URL
2. Build: `mvn clean install`
3. Post-build: Deploy WAR to container or SCP to Tomcat

---

### 6️⃣ Jenkins Pipeline (Using `Jenkinsfile`)

```groovy
pipeline {
    agent any

    stages {
        stage('Clone') {
            steps {
                git 'https://github.com/perumandlahemakumari/jenkins-tomcat-java-cicd.git'
            }
        }
        stage('Build') {
            steps {
                sh 'mvn clean install'
            }
        }
        stage('Deploy to Tomcat') {
            steps {
                sshagent (credentials: ['tomcat-server-cred']) {
                    sh '''
                    scp target/SimpleJavaWebApp.war user@your-server-ip:/opt/tomcat9/webapps/
                    '''
                }
            }
        }
    }
}
```

---

### 7️⃣ Access the App

After successful deployment, visit:

```
http://<your-server-ip>:8081/SimpleJavaWebApp/hello
```

You should see:
```
Hello from Java Web App!
```

---

## ✅ Final Notes

- Make sure Jenkins has access to Tomcat (via SCP or plugin).
- Use SSH credentials securely inside Jenkins.
- Monitor Tomcat logs for deployment errors.

---

## 👨‍💻 Author

Created with 💻 by Hemakumari - Feel free to contribute or fork!
