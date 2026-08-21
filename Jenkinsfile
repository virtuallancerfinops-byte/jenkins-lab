pipeline {
    agent any

    environment {
        IMAGE_NAME = "dockfinop/jenkins-lab"
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
                sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
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
                sh 'docker push $IMAGE_NAME:$IMAGE_TAG'
            }
        }
    }

    post {
        success {
            echo 'Docker image pushed successfully'
        }
    }
}
        stage('Deploy to Kubernetes') {
            steps {
             sshagent(credentials: ['k3s-ssh']) {
              sh '''
               ssh -o StrictHostKeyChecking=no deploy@172.31.8.222 << 'EOF'
                kubectl set image deployment/jenkins-lab \
                 web=dockfinop/jenkins-lab:${BUILD_NUMBER}

              kubectl rollout status deployment/jenkins-lab
            EOF
            '''
        }
    }
}
