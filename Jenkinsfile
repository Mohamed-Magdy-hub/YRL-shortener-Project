pipeline {
    agent {
        kubernetes {
            label 'url-shortener-pipeline'
            defaultContainer 'jnlp'
            yaml """
apiVersion: v1
kind: Pod
spec:
  nodeSelector:
    node: app  # This forces the pod to run on your 'app' node
  containers:
  - name: docker
    image: docker:24.0.2-dind
    securityContext:
      privileged: true
    volumeMounts:
    - name: docker-graph-storage
      mountPath: /var/lib/docker
  - name: kubectl
    image: kubernetes/kubectl:1.28.10  # Fixed image
    command:
    - cat
    tty: true
  volumes:
  - name: docker-graph-storage
    emptyDir: {}
"""
        }
    }

    environment {
        DOCKER_USER = credentials('dockerhub-username')  // Docker Hub username credential ID
        DOCKER_PASS = credentials('dockerhub-password')  // Docker Hub password/Token credential ID
    }

    stages {

        stage('Checkout Code') {
            steps {
                container('jnlp') {
                    git branch: 'main', url: 'https://github.com/Mohamed-Magdy-hub/YRL-shortener-Project.git'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                container('docker') {
                    sh """
                    docker build -t makenmohamed/url-shortener:latest .
                    """
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                container('docker') {
                    sh """
                    echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                    docker push makenmohamed/url-shortener:latest
                    """
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                container('kubectl') {
                    sh """
                    kubectl config set-context --current --namespace=default
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
                    """
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
