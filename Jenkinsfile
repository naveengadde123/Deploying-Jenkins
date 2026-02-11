pipeline {
    agent any

    environment {
        IMAGE_NAME = "naveengadde/jenkins-ci"
        IMAGE_TAG  = "${BUILD_NUMBER}"
        CONTAINER_NAME = "jenkins-ci-container"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                bat 'docker build -t %IMAGE_NAME%:%IMAGE_TAG% .'
            }
        }

        stage('Stop & Remove Old Container') {
            steps {
                bat 'docker rm -f %CONTAINER_NAME% || exit 0'
            }
        }

        stage('Run New Container') {
            steps {
                bat '''
                docker run -d ^
                --name %CONTAINER_NAME% ^
                -p 9090:8080 ^
                %IMAGE_NAME%:%IMAGE_TAG%
                '''
            }
        }
    }
}
