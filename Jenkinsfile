pipeline {
    agent any

    environment {
        IMAGE_NAME = "custom-jenkins"
        IMAGE_TAG  = "1.0"
        CONTAINER_NAME = "custom-jenkins-container"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
            }
        }

        stage('Stop & Remove Old Container') {
            steps {
                sh 'docker rm -f $CONTAINER_NAME || true'
            }
        }

        stage('Run New Container') {
            steps {
                sh '''
                docker run -d \
                --name $CONTAINER_NAME \
                -p 9090:8080 \
                $IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }

    }
}
