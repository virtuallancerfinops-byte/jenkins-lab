
pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                echo 'Source code already checked out by Jenkins'
                sh 'pwd'
                sh 'ls -la'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t jenkins-lab:v2 .'
            }
        }

        stage('Verify Image') {
            steps {
                sh 'docker images | grep jenkins-lab'
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully'
        }

        failure {
            echo 'Pipeline failed'
        }
    }
}
