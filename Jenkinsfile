pipeline {
 agent any // Runs on any available agent
 stages {
 stage('Build') {
 steps {
 echo "Building the project..."
 //sh 'ls -la' // Linux/macOS command
   bat 'dir'
 }
 }
 stage('Test') {
 steps {
 echo "Running tests..."
 }
 }
 stage('Deploy') {
 steps {
 echo "Deploying..."
 }
 }
 }
}
pipeline {
    agent any
    environment {
        DOCKER_IMAGE = 'dockerhub_username/my-web-app'
        DOCKER_CREDENTIALS_ID = 'docker-hub-credentials'
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
                    dockerImage = docker.build("${DOCKER_IMAGE}:latest")
                }
            }
        }
        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDENTIALS_ID) {
                        dockerImage.push('latest')
                    }
                }
            }
        }
    }
}
