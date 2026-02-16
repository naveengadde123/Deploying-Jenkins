pipeline {
    agent any

    environment {
        AWS_REGION = "ap-south-1"
        ECR_REPO   = "947754984655.dkr.ecr.ap-south-1.amazonaws.com/custom-jenkins"
        IMAGE_TAG  = "${BUILD_NUMBER}"
        CLUSTER_NAME = "jenkins-cluster"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                docker build -t $ECR_REPO:$IMAGE_TAG .
                """
            }
        }

        stage('Login to ECR') {
            steps {
                sh """
                aws ecr get-login-password --region $AWS_REGION | \
                docker login --username AWS --password-stdin 947754984655.dkr.ecr.ap-south-1.amazonaws.com
                """
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh """
                docker push $ECR_REPO:$IMAGE_TAG
                """
            }
        }

        stage('Update Kubeconfig') {
            steps {
                sh """
                aws eks update-kubeconfig --region $AWS_REGION --name $CLUSTER_NAME
                """
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh """
                sed -i 's|:1.0|:$IMAGE_TAG|g' deployment.yaml
                kubectl apply -f deployment.yaml
                kubectl apply -f service.yaml
                """
            }
        }
    }
}
