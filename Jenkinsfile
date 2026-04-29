pipeline {
    agent any
    environment {
        DEPLOY_HOST = "3.80.41.252"
        DEPLOY_USER = "ubuntu"
    }
    stages {
        stage('Clone Code') {
            steps {
                git branch: 'main',
                url: 'https://github.com/saravanakumar4247/lirw-react-node-mysql-app.git'
            }
        }
        stage('SonarQube Scan') {
            steps {
                script {
                    def scannerHome = tool 'sonar-scanner'
                    withSonarQubeEnv('sonar') {
                        sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=three-tier-app -Dsonar.sources=."
                    }
                }
            }
        }
        stage('Deploy to Docker Server') {
            steps {
                sh """
                ssh -o StrictHostKeyChecking=no ${DEPLOY_USER}@${DEPLOY_HOST} '
                rm -rf app &&
                git clone https://github.com/saravanakumar4247/lirw-react-node-mysql-app.git app &&
                cd app &&
                docker compose down || true &&
                docker compose up -d --build
                '
                """
            }
        }
    }
}
