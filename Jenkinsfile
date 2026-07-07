pipeline {
    agent any

    environment {
        IMAGE_NAME = "gsafetyweiwen/nginx-demo"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                '''
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    '''
                }
            }
        }

        stage('Push Image') {
            steps {
                sh '''
                docker push ${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                kubectl set image deployment/nginx-demo \
                nginx-demo=${IMAGE_NAME}:${IMAGE_TAG}

                kubectl rollout status deployment/nginx-demo --timeout=120s
                '''
            }
        }

    }

    post {

    success {
        echo "Deployment Successful"
    }

    failure {

        sh '''
        kubectl rollout undo deployment/nginx-demo
        '''

        echo "Rollback Completed"
    }

    }

}