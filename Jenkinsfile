pipeline { 
    agent any  // Runs on any available agent 
    stages { 
        stage('Build') { 
            steps { 
                echo "Building the project..." 
                //sh 'ls -la'  // Linux/macOS command 
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
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "cesalie0826/my-web-app"
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
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-hub-credentials') {
                        dockerImage.push('latest')
                    }
                }
            }
        }

        stage('Deploy to Remote Docker Host') {
            steps {
                sshagent(['remote-host-ssh-key']) {
                    sh '''
                        ssh ubuntu@your-server-ip "docker pull cesalie0826/my-web-app:latest"
                        ssh ubuntu@your-server-ip "docker rm -f my-web-app || true"
                        ssh ubuntu@your-server-ip "docker run -d --name my-web-app -p 8080:80 cesalie0826/my-web-app:latest"
                    '''
                }
            }
        }
    }
}
