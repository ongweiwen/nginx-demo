pipeline {
    agent any

    environment {
        IMAGE_NAME = "gsafetyweiwen/nginx-demo"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                '''
            }
        }

        stage('Verify Image') {
            steps {
                sh '''
                    docker images | grep nginx-demo
                '''
            }
        }
    }
}