pipeline {
    agent any

    environment {
        AWS_REGION   = "ap-south-1"
        ACCOUNT_ID   = "947754984655"
        ECR_REPO     = "${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/custom-jenkins"
        IMAGE_TAG    = "${BUILD_NUMBER}"
        CLUSTER_NAME = "jenkins-cluster"
        K8S_NAMESPACE = "default"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t $ECR_REPO:$IMAGE_TAG ."
            }
        }

        stage('Login to ECR') {
            steps {
                sh """
                aws ecr get-login-password --region $AWS_REGION | \
                docker login --username AWS --password-stdin ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                """
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh "docker push $ECR_REPO:$IMAGE_TAG"
            }
        }

        stage('Update Kubeconfig') {
            steps {
                sh "aws eks update-kubeconfig --region $AWS_REGION --name $CLUSTER_NAME"
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh """
                # Update image in K8s deployment.yaml dynamically
                sed -i "s|image:.*|image: $ECR_REPO:$IMAGE_TAG|g" k8s/deployment.yaml

                # Apply manifests (creates if not exists, updates if exists)
                kubectl apply -f k8s/deployment.yaml -n $K8S_NAMESPACE
                kubectl apply -f k8s/service.yaml -n $K8S_NAMESPACE

                # Wait for rollout to complete
                kubectl rollout status deployment/jenkins -n $K8S_NAMESPACE
                """
            }
        }
    }
}
