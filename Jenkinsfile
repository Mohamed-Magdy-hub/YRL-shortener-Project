pipeline {
    agent {
        kubernetes {
            yamlFile 'kaniko-pod.yaml'
        }
    }
    environment {
        DOCKER_USERNAME = credentials('dockerhub-username') // Jenkins Docker Hub username
        DOCKER_PASSWORD = credentials('dockerhub-password') // Jenkins Docker Hub password/token
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build & Push Docker Image') {
            steps {
                container('kaniko') {
                    sh """
                    /kaniko/executor \
                      --dockerfile=/workspace/Dockerfile \
                      --context=/workspace \
                      --destination=$DOCKER_USERNAME/url-shortener:latest \
                      --cache=true \
                      --registry-mirror=https://index.docker.io
                    """
                }
            }
        }
    }
    post {
        always {
            echo 'Pipeline finished'
        }
    }
}
