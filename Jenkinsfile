pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                echo 'Downloading source code from GitHub...'
            }
        }

        stage('Show Files') {
            steps {
                sh 'pwd'
                sh 'ls -la'
            }
        }

    }
}