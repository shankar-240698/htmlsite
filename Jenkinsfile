pipeline {
    agent any

    environment {
        IMAGE_NAME = 'msshankar/htmlpage:latest'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME .'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withDockerRegistry([ credentialsId: 'dockerhub-creds', url: 'https://registry.hub.docker.com' ]) {
                    sh 'docker push $IMAGE_NAME'
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

                    aws eks --region us-west-2 update-kubeconfig --name eks-cluster
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
