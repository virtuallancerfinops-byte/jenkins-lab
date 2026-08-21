pipeline {
    agent any

    environment {
        IMAGE_NAME = "dockfinop/jenkins-lab"
        IMAGE_TAG  = "${BUILD_NUMBER}"

        K3S_HOST = "172.31.8.222"
        K3S_USER = "deploy"
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
                docker build -t $IMAGE_NAME:$IMAGE_TAG .
                '''
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
                    echo "$DOCKER_PASS" | docker login \
                      -u "$DOCKER_USER" --password-stdin
                    '''
                }
            }
        }

        stage('Push Image') {
            steps {
                sh '''
                docker push $IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                # Clean old manifests
                ssh -o StrictHostKeyChecking=no $K3S_USER@$K3S_HOST \
                  "rm -rf /home/deploy/k8s"

                # Copy latest manifests
                scp -o StrictHostKeyChecking=no -r k8s \
                  $K3S_USER@$K3S_HOST:/home/deploy/

                # Update image and deploy
                ssh -o StrictHostKeyChecking=no $K3S_USER@$K3S_HOST "
                  export KUBECONFIG=/home/deploy/.kube/config

                  sed -i 's|image:.*|image: '$IMAGE_NAME':'$IMAGE_TAG'|' \
                    /home/deploy/k8s/deployment.yaml

                  kubectl apply -f /home/deploy/k8s/

                  kubectl rollout status deployment/jenkins-lab
                "
                '''
            }
        }
    }

    post {
        success {
            echo "CI/CD Pipeline completed successfully"
        }

        failure {
            echo "Pipeline failed"
        }

        always {
            sh 'docker logout || true'
        }
    }
}
