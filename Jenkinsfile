pipeline {
    agent {
        kubernetes {
            label 'url-shortener-pipeline'
            defaultContainer 'jnlp'
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: docker
    image: docker:24.0.2-dind
    securityContext:
      privileged: true
    volumeMounts:
    - name: docker-graph-storage
      mountPath: /var/lib/docker
  - name: kubectl
    image: bitnami/kubectl:1.28
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
        IMAGE_NAME = 'makenmohamed/url-shortener:latest'
        KUBE_NAMESPACE = 'default'   // Change if your app uses another namespace
    }

    stages {
        stage('Checkout Code') {
            steps {
                git url: 'https://github.com/Mohamed-Magdy-hub/YRL-shortener-Project.git', branch: 'main'
            }
        }

        stage('Build Docker Image') {
            steps {
                container('docker') {
                    sh """
                    docker build -t $IMAGE_NAME .
                    """
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                container('docker') {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh """
                        docker logout
                        echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                        docker push $IMAGE_NAME
                        """
                    }
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                container('kubectl') {
                    sh """
                    kubectl config use-context <YOUR_EKS_CONTEXT>  # e.g., aws eks --region us-east-1 update-kubeconfig --name depi-cluster
                    kubectl set image deployment/url-shortener url-shortener=$IMAGE_NAME -n $KUBE_NAMESPACE
                    kubectl rollout status deployment/url-shortener -n $KUBE_NAMESPACE
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check logs!'
        }
    }
}
