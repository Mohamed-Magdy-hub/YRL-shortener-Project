pipeline {
    agent {
        kubernetes {
            label 'kaniko-agent'
            defaultContainer 'jnlp'
            yamlFile 'kaniko-pod.yaml' // path to your static pod YAML
        }
    }

    environment {
        DOCKER_REGISTRY = 'docker.io/makenmohamed' // your Docker Hub username
        DOCKER_REPO     = 'yrl-shortener'          // your repo name
        IMAGE_TAG       = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Mohamed-Magdy-hub/YRL-shortener-Project.git',
                    credentialsId: 'dockerhub-credentials'
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
                      --insecure
                    """
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline finished"
        }
    }
}
