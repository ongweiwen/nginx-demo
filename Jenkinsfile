pipeline {
    agent any

    parameters {
        choice(
            name: 'DEPLOY_ENV',
            choices: ['DEV', 'UAT', 'PROD'],
            description: 'Select the Kubernetes environment to deploy'
        )
    }

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

        stage('Deploy Selected Environment') {
            steps {
                script {
                    def namespace = params.DEPLOY_ENV.toLowerCase()
                    def valuesFile = "./nginx-demo-chart/values-${namespace}.yaml"
                    def releaseName = "nginx-demo-${namespace}"
                    def deploymentName = "nginx-demo-${namespace}-nginx-demo-chart"

                    echo "Deploying to ${params.DEPLOY_ENV}"
                    echo "Namespace: ${namespace}"
                    echo "Helm release: ${releaseName}"
                    echo "Image: ${IMAGE_NAME}:${IMAGE_TAG}"

                    sh """
                        helm upgrade --install ${releaseName} \
                            ./nginx-demo-chart \
                            -n ${namespace} \
                            --create-namespace \
                            -f ${valuesFile} \
                            --set image.repository=${IMAGE_NAME} \
                            --set image.tag=${IMAGE_TAG}

                        kubectl rollout status \
                            deployment/${deploymentName} \
                            -n ${namespace} \
                            --timeout=120s
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Deployment to ${params.DEPLOY_ENV} successful"
        }

        failure {
            echo "Pipeline failed"
        }
    }
}
