pipeline {
    agent any

    environment {
        REGISTRY = "saravana4247"   // 🔴 CHANGE THIS
        IMAGE_BACKEND = "app-backend"
        IMAGE_FRONTEND = "app-frontend"
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
                        sh """
                        ${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=three-tier-app \
                        -Dsonar.sources=.
                        """
                    }
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                sh '''
                docker build -t $REGISTRY/$IMAGE_BACKEND:latest -f Dockerfile.backend .
                docker build -t $REGISTRY/$IMAGE_FRONTEND:latest -f Dockerfile.frontend .
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push $REGISTRY/$IMAGE_BACKEND:latest
                    docker push $REGISTRY/$IMAGE_FRONTEND:latest
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh '''
                    kubectl apply -f k8s/

                    kubectl rollout restart deployment backend
                    kubectl rollout restart deployment frontend

                    kubectl rollout status deployment backend
                    kubectl rollout status deployment frontend
                    '''
                }
            }
        }
    }
}
