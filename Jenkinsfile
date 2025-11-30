pipeline {
    agent {
        kubernetes {
            label 'kaniko-agent' // must match the label in your kaniko-pod.yaml
        }
    }
    environment {
        DOCKER_REGISTRY = "docker.io"
        DOCKER_REPO     = "makenmohamed/YRL-shortener-Project"
        IMAGE_TAG       = "${env.BUILD_NUMBER}"
    }
    stages {
        stage('Checkout') {
            steps {
                echo "Cloning repository..."
                checkout scm
            }
        }
        stage('Build & Push Docker Image') {
            steps {
                container('kaniko') {
                    echo "Building Docker image with Kaniko..."
                    sh """
                    /kaniko/executor \
                      --dockerfile=\$WORKSPACE/Dockerfile \
                      --context=\$WORKSPACE \
                      --destination=\$DOCKER_REGISTRY/\$DOCKER_REPO:\$IMAGE_TAG \
                      --destination=\$DOCKER_REGISTRY/\$DOCKER_REPO:latest \
                      --verbosity=info
                    """
                }
            }
        }
    }
    post {
        always {
            echo "Pipeline finished."
        }
        success {
            echo "Docker image successfully built and pushed!"
        }
        failure {
            echo "Pipeline failed. Check logs."
        }
    }
}
