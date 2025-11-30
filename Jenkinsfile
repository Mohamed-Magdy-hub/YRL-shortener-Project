pipeline {
    agent {
        kubernetes {
            yamlFile 'kaniko-pod.yaml'
        }
    }
    environment {
        DOCKER_USERNAME = credentials('dockerhub-username') 
        DOCKER_PASSWORD = credentials('dockerhub-password') 
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build & Push Docker Image') {
            steps {
                script {
                    if (fileExists('Dockerfile')) {
                        container('kaniko') {
                            sh """
                            /kaniko/executor \
                              --dockerfile=/workspace/Dockerfile \
                              --context=/workspace \
                              --destination=$DOCKER_USERNAME/url-shortener:latest \
                              --cache=true
                            """
                        }
                    } else {
                        echo "Dockerfile not found. Skipping Kaniko build."
                    }
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
