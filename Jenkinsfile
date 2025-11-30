pipeline {
    agent {
        kubernetes {
            label 'kaniko-agent'
            defaultContainer 'jnlp'
            yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    some-label: kaniko-agent
spec:
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:latest
    command:
      - /busybox/sh
    args:
      - -c
      - "sleep 99d"
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
        IMAGE = "makenmohamed/url-shortener:latest"
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
                        --context=\$WORKSPACE \
                        --destination=${IMAGE} \
                        --skip-tls-verify
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ Docker image built and pushed: ${IMAGE}"
        }
        failure {
            echo "❌ Pipeline failed"
        }
    }
}
