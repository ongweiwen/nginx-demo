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
                    docker build \
                        -t ${IMAGE_NAME}:${IMAGE_TAG} .
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
                        echo "$DOCKER_PASS" | \
                        docker login -u "$DOCKER_USER" --password-stdin
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

        stage('Deploy DEV') {
            steps {
                sh '''
                    helm upgrade --install nginx-demo-dev \
                        ./nginx-demo-chart \
                        -n dev \
                        --create-namespace \
                        -f ./nginx-demo-chart/values-dev.yaml \
                        --set image.repository=${IMAGE_NAME} \
                        --set image.tag=${IMAGE_TAG}

                    kubectl rollout status \
                        deployment/nginx-demo-dev-nginx-demo-chart \
                        -n dev \
                        --timeout=120s
                '''
            }
        }

        stage('Deploy UAT') {
            steps {
                input message: 'Deploy to UAT?'

                sh '''
                    helm upgrade --install nginx-demo-uat \
                        ./nginx-demo-chart \
                        -n uat \
                        --create-namespace \
                        -f ./nginx-demo-chart/values-uat.yaml \
                        --set image.repository=${IMAGE_NAME} \
                        --set image.tag=${IMAGE_TAG}
                '''
            }
        }

        stage('Deploy PROD') {
            steps {
                input message: 'Deploy to PROD?'

                sh '''
                    helm upgrade --install nginx-demo-prod \
                        ./nginx-demo-chart \
                        -n prod \
                        --create-namespace \
                        -f ./nginx-demo-chart/values-prod.yaml \
                        --set image.repository=${IMAGE_NAME} \
                        --set image.tag=${IMAGE_TAG}
                '''
            }
        }
    }
}