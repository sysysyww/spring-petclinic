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


        stage('Deploy to Production') {
            steps {
                script {
                    def jarPath = sh(script: "readlink -f target/spring-petclinic-*.jar", returnStdout: true).trim()
                    sh "ansible-playbook -i deploy/inventory.ini deploy/deploy.yml --extra-vars 'jar_path=${jarPath}'"
                }
            }
        }

        stage('Security Scan') {
            steps {
                sh '''
                    curl "http://zap:8080/JSON/ascan/action/scan/?url=http://prod-server:8080&recurse=true&inScopeOnly=false"
                    sleep 20
                    curl "http://zap:8080/OTHER/core/other/htmlreport/" -o zap-report.html
                '''
            }
        }

        stage('Publish ZAP Report') {
            steps {
                publishHTML(target: [
                    allowMissing: false,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: '.',
                    reportFiles: 'zap-report.html',
                    reportName: 'OWASP ZAP HTML Report'
                ])
            }
        }

    }

    triggers {
        pollSCM('* * * * *') 
    }
}