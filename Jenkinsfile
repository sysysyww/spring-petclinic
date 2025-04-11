pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/sysysyww/spring-petclinic.git'
            }
        }

        stage('Build') {
            steps {
                sh './mvnw clean package'
            }
        }

        stage('Static Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh './mvnw sonar:sonar'
                }
            }
        }

        stage('Security Scan') {
            steps {
                sh 'curl "http://zap:8080/JSON/ascan/action/scan/?url=http://localhost:8080&recurse=true&inScopeOnly=false"'
            }
        }

        stage('Deploy to Production') {
            steps {
                sh 'ansible-playbook -i deploy/inventory.ini deploy/deploy.yml'
            }
        }
    }
}