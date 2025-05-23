pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'dockerhub-creds'  // Jenkins credentials ID for Docker Hub
        IMAGE_NAME = 'mssankar/basic-html-app'
        IMAGE_TAG = 'latest'
        KUBECONFIG_PATH = '/home/jenkins/.kube/config'  // Path to kubeconfig on Jenkins agent
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', DOCKERHUB_CREDENTIALS) {
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                script {
                    // Use kubectl with kubeconfig to deploy the app
                    sh """
                    export KUBECONFIG=${KUBECONFIG_PATH}
                    kubectl apply -f deployment.yaml
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Build or deployment failed.'
        }
    }
}
