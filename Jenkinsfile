pipeline {
    agent {
        kubernetes {
            yamlFile 'jenkins-dind-pod.yaml'
            defaultContainer 'docker'
        }
    }
    environment {
        DOCKER_IMAGE = "makenmohamed/url-shortener:latest"
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
    }
}
