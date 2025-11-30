pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'dockerhub-credentials'
        IMAGE_NAME = 'makenmohamed/url-shortener:latest'
        KUBECONFIG_CREDENTIALS_ID = 'eks-kubeconfig'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Mohamed-Magdy-hub/YRL-shortener-Project.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t $IMAGE_NAME ."
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: env.DOCKER_CREDENTIALS_ID,
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh "docker push $IMAGE_NAME"
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                withCredentials([file(credentialsId: env.KUBECONFIG_CREDENTIALS_ID, variable: 'KUBECONFIG_FILE')]) {
                    sh 'export KUBECONFIG=$KUBECONFIG_FILE'
                    sh 'kubectl apply -f k8s/deployment.yaml'
                    sh 'kubectl rollout status deployment/url-shortener-deployment'
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline finished."
        }
    }
}
