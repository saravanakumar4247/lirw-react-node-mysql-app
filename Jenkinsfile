pipeline {
    agent any
    environment {
        DEPLOY_HOST = "3.80.41.252"
        DEPLOY_USER = "ubuntu"
        IMAGE_NAME = "three-tier-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
        K8S_NAMESPACE = "default"
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
                sshagent(credentials: ['docker-server-key']) {
                    sh '''
                    ssh -tt -o StrictHostKeyChecking=no ubuntu@3.80.41.252 << EOF
                    whoami
                    docker --version
                    docker compose version
                    rm -rf app
                    git clone https://github.com/saravanakumar4247/lirw-react-node-mysql-app.git app
                    cd app
                    docker compose down || true
                    docker compose up -d --build
                    exit
EOF
                    '''
                }
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockerhub-creds', url: '') {
                        sh """
                            docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                            docker tag ${IMAGE_NAME}:${IMAGE_TAG} your-dockerhub-username/${IMAGE_NAME}:${IMAGE_TAG}
                            docker push your-dockerhub-username/${IMAGE_NAME}:${IMAGE_TAG}
                        """
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh '''
                        # Apply all manifests inside k8s/ folder
                        kubectl apply -f k8s/

                        # Wait for backend rollout to complete
                        kubectl rollout status deployment/backend

                        # Wait for frontend rollout to complete
                        kubectl rollout status deployment/frontend
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline succeeded! App deployed to Kubernetes."
        }
        failure {
            echo "❌ Pipeline failed! Check the logs above."
        }
    }
}
