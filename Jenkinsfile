pipeline {
    agent {
        kubernetes {
            label 'kaniko-agent'
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
    command:
    - cat
    tty: true
    volumeMounts:
    - name: kaniko-secret
      mountPath: /kaniko/.docker
  restartPolicy: Never
  volumes:
  - name: kaniko-secret
    secret:
      secretName: dockerhub-credentials
"""
        }
    }

    environment {
        DOCKER_IMAGE = "makenmohamed/yrl-shortener-project" // replace with your Docker Hub repo
        DOCKER_TAG = "latest"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Mohamed-Magdy-hub/YRL-shortener-Project.git'
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                container('kaniko') {
                    sh """
                    /kaniko/executor \
                      --dockerfile=Dockerfile \
                      --context=./ \
                      --destination=${DOCKER_IMAGE}:${DOCKER_TAG} \
                      --cleanup
                    """
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline finished!"
        }
        success {
            echo "Docker image pushed successfully!"
        }
        failure {
            echo "Pipeline failed."
        }
    }
}
