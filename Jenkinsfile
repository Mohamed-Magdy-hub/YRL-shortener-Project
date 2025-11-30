pipeline {
    agent {
        kubernetes {
            label 'url-shortener-pipeline'
            defaultContainer 'docker'
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
    - name: workspace-volume
      mountPath: /home/jenkins/agent
  - name: jnlp
    image: jenkins/inbound-agent:latest
    volumeMounts:
    - name: docker-graph-storage
      mountPath: /var/lib/docker
    - name: workspace-volume
      mountPath: /home/jenkins/agent
  volumes:
  - name: docker-graph-storage
    emptyDir: {}
  - name: workspace-volume
    emptyDir: {}
"""
        }
    }

    environment {
        DOCKER_USER = 'makenmohamed'
        DOCKER_PASS = credentials('dockerhub-credentials') // Jenkins stored password
        KUBECONFIG = '/home/jenkins/.kube/config'          // mount or set kubeconfig
    }

    stages {
        stage('Checkout Code') {
            steps {
                container('docker') {
                    checkout scm
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                container('docker') {
                    sh 'docker build -t $DOCKER_USER/url-shortener:latest .'
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                container('docker') {
                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push $DOCKER_USER/url-shortener:latest
                    '''
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                container('docker') {
                    sh '''
                    # Install kubectl in container
                    curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
                    chmod +x kubectl
                    mv kubectl /usr/local/bin/

                    # Deploy app
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml

                    # Wait for pod ready
                    kubectl wait --for=condition=ready pod -l app=url-shortener --timeout=120s

                    # Show status
                    kubectl get pods -o wide
                    kubectl get svc url-shortener-service
                    '''
                }
            }
        }
    }
}
