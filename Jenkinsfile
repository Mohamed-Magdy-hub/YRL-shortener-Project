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
    image: bitnami/kubectl:1.28.6
    command:
    - cat
    tty: true
  - name: jnlp
    image: jenkins/inbound-agent:latest
    env:
    - name: JENKINS_SECRET
      value: "\${JENKINS_SECRET}"
    - name: JENKINS_TUNNEL
      value: "\${JENKINS_TUNNEL}"
    - name: JENKINS_AGENT_NAME
      value: "\${JENKINS_AGENT_NAME}"
  volumes:
  - name: docker-graph-storage
    emptyDir: {}
"""
        }
    }

    environment {
        IMAGE_NAME = "makenmohamed/url-shortener"
        IMAGE_TAG = "latest"
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout([$class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[url: 'https://github.com/Mohamed-Magdy-hub/YRL-shortener-Project.git', credentialsId: 'dockerhub-credentials']]
                ])
            }
        }

        stage('Build Docker Image') {
            steps {
                container('docker') {
                    sh """
                        docker build -t \$IMAGE_NAME:\$IMAGE_TAG .
                    """
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                container('docker') {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh """
                            echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                            docker push \$IMAGE_NAME:\$IMAGE_TAG
                        """
                    }
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                container('kubectl') {
                    withKubeConfig([credentialsId: 'eks-kubeconfig']) {
                        sh """
                            kubectl apply -f k8s/deployment.yaml
                            kubectl rollout status deployment/url-shortener
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished.'
        }
        success {
            echo 'Deployment succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
