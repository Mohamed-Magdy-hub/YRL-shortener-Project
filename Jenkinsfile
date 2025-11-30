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
  - name: jnlp
    image: jenkins/inbound-agent:latest
    resources:
      requests:
        memory: "256Mi"
        cpu: "100m"
  - name: docker
    image: docker:24.0.2-dind
    securityContext:
      privileged: true
    volumeMounts:
      - mountPath: /var/lib/docker
        name: docker-graph-storage
  - name: kubectl
    image: bitnami/kubectl:latest
  volumes:
  - emptyDir: {}
    name: docker-graph-storage
"""
        }
    }

    environment {
        DOCKER_USER = 'makenmohamed'
        DOCKER_PASS = credentials('dockerhub-credentials')
        IMAGE_NAME = 'makenmohamed/url-shortener:latest'
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
                    sh """
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push $IMAGE_NAME
                    """
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                container('kubectl') {
                    sh """
                    cat <<EOF > deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: url-shortener
spec:
  replicas: 1
  selector:
    matchLabels:
      app: url-shortener
  template:
    metadata:
      labels:
        app: url-shortener
    spec:
      nodeSelector:
        type: app
      containers:
      - name: url-shortener
        image: $IMAGE_NAME
        ports:
        - containerPort: 3000
EOF

                    cat <<EOF > service.yaml
apiVersion: v1
kind: Service
metadata:
  name: url-shortener-service
spec:
  type: LoadBalancer
  selector:
    app: url-shortener
  ports:
    - port: 80
      targetPort: 3000
EOF

                    kubectl apply -f deployment.yaml
                    kubectl apply -f service.yaml
                    kubectl wait --for=condition=ready pod -l app=url-shortener --timeout=120s
                    kubectl get pods -o wide
                    kubectl get svc url-shortener-service
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline finished successfully! Check the LoadBalancer IP for your app."
        }
        failure {
            echo "Pipeline failed! Check logs for errors."
        }
    }
}
