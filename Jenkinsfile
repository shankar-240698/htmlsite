pipeline {
    agent any

    environment {
        IMAGE_NAME = 'msshankar/htmlpage'
        IMAGE_NAME = 'msshankar/htmlpage'
        IMAGE_TAG = 'latest'
        KUBECONFIG_PATH = '/home/jenkins/.kube/config'
        CLUSTER_NAME = 'eks-cluster'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                """
            }
        }

       stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: DOCKERHUB_CREDENTIALS, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    docker push ${IMAGE_NAME}:${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-creds']]) {
                    sh '''
                    export AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID
                    export AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY
                    export KUBECONFIG=/home/jenkins/.kube/config

                    aws eks --region us-west-2 update-kubeconfig --name $CLUSTER_NAME
                    kubectl apply -f deployment.yaml
                    '''
                }
            }
        }
    }

    post {
        failure {
            echo 'Build or deployment failed.'
        }
    }
}
