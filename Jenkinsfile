```groovy
pipeline {
    agent any

    environment {
        IMAGE_NAME = "gsafetyweiwen/nginx-demo"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

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
                        echo "$DOCKER_PASS" | docker login \
                            -u "$DOCKER_USER" \
                            --password-stdin
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
    }

    post {
        success {
            echo "DEV deployment successful"
        }

        failure {
            echo "Pipeline failed"
        }
    }
}
```
