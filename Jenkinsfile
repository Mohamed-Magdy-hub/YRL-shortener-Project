pipeline {
    agent {
        kubernetes {
            yamlFile 'jenkins-dind-kubectl-pod.yaml'
            defaultContainer 'docker'
        }
    }
    environment {
        DOCKER_IMAGE = "makenmohamed/url-shortener:latest"
        KUBE_NAMESPACE = "app"
        DEPLOYMENT_NAME = "url-shortener"
    }
    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/Mohamed-Magdy-hub/YRL-shortener-Project.git', branch: 'main'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker version'
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh 'docker push $DOCKER_IMAGE'
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                container('kubectl') {
                    sh '''
                    export KUBECONFIG=/root/.kube/config
                    
                    # Wait for kubeconfig to be mounted
                    if [ ! -f "$KUBECONFIG" ]; then
                        echo "Error: kubeconfig not found at $KUBECONFIG"
                        exit 1
                    fi
                    
                    # Test kubectl connection
                    kubectl cluster-info
                    
                    # Update deployment if exists, else create
                    if kubectl get deployment ${DEPLOYMENT_NAME} -n ${KUBE_NAMESPACE} 2>/dev/null; then
                        echo "Updating existing deployment..."
                        kubectl set image deployment/${DEPLOYMENT_NAME} ${DEPLOYMENT_NAME}=${DOCKER_IMAGE} -n ${KUBE_NAMESPACE}
                    else
                        echo "Creating new deployment..."
                        kubectl create deployment ${DEPLOYMENT_NAME} --image=${DOCKER_IMAGE} -n ${KUBE_NAMESPACE}
                        kubectl expose deployment ${DEPLOYMENT_NAME} --type=LoadBalancer --port=80 --target-port=3000 -n ${KUBE_NAMESPACE}
                    fi
                    
                    # Wait for rollout to complete
                    kubectl rollout status deployment/${DEPLOYMENT_NAME} -n ${KUBE_NAMESPACE} --timeout=5m
                    '''
                }
            }
        }
    }
}
