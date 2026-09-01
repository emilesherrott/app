pipeline {
    agent any

    environment {
        IMAGE_NAME = "emilesherrott/hello-pipeline-app"
        IMAGE_TAG  = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo "Building ${IMAGE_NAME}:${IMAGE_TAG}"
            }
        }

        stage('Install & Test') {
            agent {
                docker { image 'node:20' }
            }
            steps {
                dir('app') {
                    sh 'npm install'
                    sh 'npm test'
                }
            }
        }

        stage('Build Image') {
            steps {
                dir('app') {
                    sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
                }
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh 'docker push $IMAGE_NAME:$IMAGE_TAG'
                }
            }
        }
    }

    post {
        success {
            echo "Done! Pushed ${IMAGE_NAME}:${IMAGE_TAG} to Docker Hub."
        }
        failure {
            echo 'Pipeline failed — check which stage went red in the Stage View.'
        }
    }
}