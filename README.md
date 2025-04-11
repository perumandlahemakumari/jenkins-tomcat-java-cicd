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

---

## 📦 Nexus Repository Setup (Artifact Management)

To store your build artifacts (WAR files) in a Nexus repository before deployment, follow these steps to set up Nexus on an EC2 instance.

---

### ☁️ Launch Nexus EC2 Instance

- **Instance Type**: `t2.medium`
- **AMI**: Amazon Linux 2 (Kernel 5.10)
- **Storage**: 20–25 GB
- **Security Group**: Open port `8081` to access the Nexus UI

---

### ⚙️ Nexus Installation Steps

#### 1️⃣ Install Java 17

```bash
sudo yum install java-17-amazon-corretto -y
```

#### 2️⃣ Set Up Directories and Download Nexus

```bash
mkdir /app && cd /app
wget -O nexus.tar.gz https://download.sonatype.com/nexus/3/nexus-3.71.0-06-unix.tar.gz
tar -xvf nexus.tar.gz
mv nexus-3* nexus
```

#### 3️⃣ Create Nexus User and Set Permissions

```bash
useradd nexus
chown -R nexus:nexus /app/nexus
chown -R nexus:nexus /app/sonatype-work
```

#### 4️⃣ Set Nexus to Run as `nexus` User

Edit `/app/nexus/bin/nexus.rc`:

```bash
vim /app/nexus/bin/nexus.rc
```

Add or modify:

```bash
run_as_user="nexus"
```

#### 5️⃣ Create Nexus Systemd Service

```bash
vim /etc/systemd/system/nexus.service
```

Paste the following:

```ini
[Unit]
Description=nexus service
After=network.target

[Service]
Type=forking
LimitNOFILE=65536
User=nexus
Group=nexus
ExecStart=/app/nexus/bin/nexus start
ExecStop=/app/nexus/bin/nexus stop
Restart=on-abort

[Install]
WantedBy=multi-user.target
```

#### 6️⃣ Enable and Start Nexus

```bash
systemctl daemon-reexec
systemctl enable nexus
systemctl start nexus
```

#### 7️⃣ Access Nexus Web Interface

Visit:  
```
http://<your-ec2-ip>:8081
```

Default credentials:  
- **Username**: `admin`  
- **Password**: Check in `/app/sonatype-work/nexus3/admin.password`

---
---

### 🔗 Integrate Nexus with Jenkins

Once Nexus is running, you can configure Jenkins to push your WAR artifacts after the build.

#### 🧩 Install Jenkins Plugin

1. Go to **Jenkins Dashboard → Manage Jenkins → Manage Plugins**
2. Under **Available Plugins**, search for:
   - ✅ `Nexus Artifact Uploader`
3. Install and restart Jenkins if needed

#### 🛠️ Job Configuration

1. Go to your Jenkins job → **Configure**
2. Under **Build Steps**, click **Add build step** → `Nexus Artifact Uploader`
3. Fill in:
   - **Nexus Version**: `Nexus3`
   - **Protocol**: `http`
   - **Nexus URL**: `<public-ip>:8081` (without `http://`)
   - **Credentials**: Add and select Nexus admin credentials
   - **Group ID**: e.g. `com.example`
   - **Artifact ID**: e.g. `SimpleJavaWebApp`
   - **Version**: e.g. `1.0.1` *(should match `pom.xml`)*
   - **Repository**: Create a Maven (hosted) repository in Nexus (e.g. `maven-releases`)
     - Type: `maven2 (hosted)`
     - Version Policy: `Release`
     - Deployment Policy: `Allow Redeploy`
   - **Packaging**: `war`
   - **File**: `target/SimpleJavaWebApp.war`

4. Click **Save**.

---

### 🚀 Deploy Updated Artifact

- Every time you update code:
  1. Bump the version in `pom.xml`
  2. Commit and push
  3. Build the Jenkins job again
  4. Check Nexus → Your WAR will be available under the specified repository



## ✅ Final Notes

- Make sure Jenkins has access to Tomcat (via SCP or plugin).
- Use SSH credentials securely inside Jenkins.
- Monitor Tomcat logs for deployment errors.

---

## 👨‍💻 Author

Created with 💻 by Hemakumari - Feel free to contribute or fork!
