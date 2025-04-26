pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'somyaagrawal/my-node-app'
        DOCKER_TAG = 'latest'    // Add this missing variable
        DOCKER_CREDENTIALS_ID = 'dockerhub'  // Jenkins credentials ID for Docker Hub login
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'master', url: 'https://github.com/somyaagarwal20/WebApiJenkins.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_CREDENTIALS_ID}") {
                        dockerImage.push("${DOCKER_TAG}")   // corrected here
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Docker image pushed successfully!'
        }
        failure {
            echo 'Build failed.'
        }
    }
}
