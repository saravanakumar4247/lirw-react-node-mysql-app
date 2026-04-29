pipeline {
    agent any

    environment {
        DEPLOY_HOST = "3.80.41.252"
        DEPLOY_USER = "ubuntu"
        APP_NAME = "three-tier-app"
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
                withSonarQubeEnv('sonar') {
                    sh '''
                    sonar-scanner \
                    -Dsonar.projectKey=three-tier-app \
                    -Dsonar.sources=. \
                    '''
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
                docker rm -f ${APP_NAME} || true &&
                docker build -t ${APP_NAME} . &&
                docker run -d -p 80:80 --name ${APP_NAME} ${APP_NAME}
                '
                """
            }
        }
    }
}
