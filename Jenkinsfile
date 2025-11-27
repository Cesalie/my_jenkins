// Pipeline 1: Basic Build, Test, Deploy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo "Building the project..."
                // Use sh for Linux/macOS or bat for Windows
                script {
                    if (isUnix()) {
                        sh 'ls -la'
                    } else {
                        bat 'dir'
                    }
                }
            }
        }
        stage('Test') {
            steps {
                echo "Running tests..."
                // Add your test commands here
                // sh 'npm test' or bat 'npm test'
            }
        }
        stage('Deploy') {
            steps {
                echo "Deploying..."
                // Add your deploy commands here
            }
        }
    }
}

// ============================================================

// Pipeline 2: Docker Build and Push (Generic Template)
pipeline {
    agent any
    environment {
        DOCKER_IMAGE = 'dockerhub_username/my-web-app'
        DOCKER_CREDENTIALS_ID = 'docker-hub-credentials'
        REGISTRY_URL = 'https://index.docker.io/v1/'
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
                    docker.withRegistry("${REGISTRY_URL}", "${DOCKER_CREDENTIALS_ID}") {
                        dockerImage.push('latest')
                        dockerImage.push("${BUILD_NUMBER}")
                    }
                }
            }
        }
    }
}

// ============================================================

// Pipeline 3: Complete CI/CD with Docker Build, Push, and Deploy
pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "cesalie0826/my-web-app"
        DOCKER_CREDENTIALS_ID = 'docker-hub-credentials'
        REGISTRY_URL = 'https://index.docker.io/v1/'
        SSH_CREDENTIALS_ID = 'remote-host-ssh-key'
        REMOTE_USER = 'ubuntu'
        REMOTE_HOST = 'your-server-ip'
        CONTAINER_NAME = 'my-web-app'
        CONTAINER_PORT = '8080'
        APP_PORT = '80'
    }
    stages {
        stage('Checkout') {
            steps {
                echo "Checking out source code..."
                checkout scm
            }
        }
        stage('Build Docker Image') {
            steps {
                echo "Building Docker image..."
                script {
                    dockerImage = docker.build("${DOCKER_IMAGE}:latest")
                    dockerImage = docker.build("${DOCKER_IMAGE}:${BUILD_NUMBER}")
                }
            }
        }
        stage('Push to Docker Hub') {
            steps {
                echo "Pushing image to Docker Hub..."
                script {
                    docker.withRegistry("${REGISTRY_URL}", "${DOCKER_CREDENTIALS_ID}") {
                        dockerImage.push('latest')
                        dockerImage.push("${BUILD_NUMBER}")
                    }
                }
            }
        }
        stage('Deploy to Remote Docker Host') {
            steps {
                echo "Deploying to remote server..."
                sshagent(["${SSH_CREDENTIALS_ID}"]) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${REMOTE_HOST} << EOF
                        
                        # Pull the latest image
                        docker pull ${DOCKER_IMAGE}:latest
                        
                        # Stop and remove existing container if running
                        docker rm -f ${CONTAINER_NAME} || true
                        
                        # Run new container
                        docker run -d \
                          --name ${CONTAINER_NAME} \
                          -p ${CONTAINER_PORT}:${APP_PORT} \
                          --restart unless-stopped \
                          ${DOCKER_IMAGE}:latest
                        
                        # Verify deployment
                        docker ps | grep ${CONTAINER_NAME}
                        
EOF
                    '''
                }
            }
        }
    }
    post {
        success {
            echo "✓ Pipeline executed successfully!"
        }
        failure {
            echo "✗ Pipeline failed. Check logs for details."
        }
        always {
            // Cleanup
            cleanWs()
        }
    }
}
