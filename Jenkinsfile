pipeline {
    agent {
        kubernetes {
            yamlFile 'kaniko-pod.yaml'
        }
    }
    environment {
        DOCKER_HUB_REPO = "YOUR_DOCKER_HUB_USERNAME/url-shortener"
        DOCKER_HUB_CREDENTIALS = "dockerhub-credentials"
    }
    stages {
        stage('Checkout') {
            steps {
                echo 'Cloning repository...'
                checkout scm
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                container('kaniko') {
                    echo 'Building Docker image with Kaniko...'
                    sh """
                    /kaniko/executor \
                        --dockerfile=/workspace/Dockerfile \
                        --destination=${DOCKER_HUB_REPO}:latest \
                        --context=/workspace \
                        --cache=true
                    """
                }
            }
        }
    }
    post {
        success {
            echo 'Pipeline finished successfully!'
        }
        failure {
            echo 'Pipeline failed. Check logs!'
        }
    }
}
