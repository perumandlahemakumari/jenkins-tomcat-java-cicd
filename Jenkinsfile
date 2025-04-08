pipeline {
    agent any

    stages {
        stage('Clone') {
            steps {
                git 'https://github.com/your-username/jenkins-tomcat-java-cicd.git'
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
