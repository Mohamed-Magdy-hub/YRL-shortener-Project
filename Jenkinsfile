pipeline {
    agent {
        kubernetes {
            yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: kaniko-agent
spec:
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:latest
    tty: true
    volumeMounts:
      - name: docker-config
        mountPath: /kaniko/.docker
  - name: jnlp
    image: jenkins/inbound-agent:latest
  volumes:
    - name: docker-config
      secret:
        secretName: dockerhub-credentials
"""
        }
    }
    environment {
        DOCKER_REGISTRY = "docker.io"
        DOCKER_REPO = "makenmohamed/url-shortener"
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build Docker Image') {
            steps {
                container('kaniko') {
                    sh """
                    /kaniko/executor \
                      --dockerfile=Dockerfile \
                      --context=\$WORKSPACE \
                      --destination=\$DOCKER_REGISTRY/\$DOCKER_REPO:\$IMAGE_TAG \
                      --verbosity=info
                    """
                }
            }
        }
        stage('Push Latest Tag') {
            steps {
                container('kaniko') {
                    sh """
                    /kaniko/executor \
                      --dockerfile=Dockerfile \
                      --context=\$WORKSPACE \
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
            echo "Docker image built and pushed successfully!"
        }
        failure {
            echo "Pipeline failed."
        }
    }
}
