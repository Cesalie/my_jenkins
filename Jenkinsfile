// ============================================================
// JENKINSFILE - Place this in root of your GitHub repository
// Filename: Jenkinsfile (no extension)
// ============================================================

pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = "cesalie0826/my-web-app"
        DOCKER_CREDENTIALS_ID = 'docker-hub-credentials'
        REGISTRY_URL = 'https://index.docker.io/v1/'
        SSH_CREDENTIALS_ID = 'remote-host-ssh-key'
        REMOTE_USER = 'ubuntu'
        REMOTE_HOST = '192.168.1.100'  // ← UPDATE THIS
        CONTAINER_NAME = 'my-web-app'
        CONTAINER_PORT = '8080'
        APP_PORT = '80'
        GITHUB_REPO = 'https://github.com/yourusername/your-repo'  // ← UPDATE THIS
    }
    
    options {
        timestamps()
        timeout(time: 45, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '20'))
        gitLabConnection('GitLab')
    }
    
    triggers {
        githubPush()  // Triggers on every push to GitHub
    }
    
    stages {
        stage('Pipeline Start') {
            steps {
                echo ""
                echo "╔════════════════════════════════════════════════════╗"
                echo "║   GITHUB → JENKINS → DOCKER → DEPLOYMENT           ║"
                echo "╚════════════════════════════════════════════════════╝"
                echo ""
                echo "═══════════════════════════════════════════════════════"
                echo "JOB INFORMATION"
                echo "═══════════════════════════════════════════════════════"
                echo "Job Name: ${env.JOB_NAME}"
                echo "Build Number: ${env.BUILD_NUMBER}"
                echo "Build URL: ${env.BUILD_URL}"
                echo "Git Branch: ${env.GIT_BRANCH}"
                echo "Git Commit: ${env.GIT_COMMIT}"
                echo "Workspace: ${env.WORKSPACE}"
                echo "Timestamp: ${new Date()}"
                echo ""
            }
        }
        
        stage('Checkout from GitHub') {
            steps {
                echo ""
                echo "═══════════════════════════════════════════════════════"
                echo "STAGE: CHECKOUT FROM GITHUB"
                echo "═══════════════════════════════════════════════════════"
                echo "[Checkout] Cloning repository from GitHub..."
                echo "[Checkout] Repository: ${GITHUB_REPO}"
                
                script {
                    try {
                        checkout([
                            $class: 'GitSCM',
                            branches: [[name: '*/main']],  // or '*/master' depending on your branch
                            extensions: [
                                [$class: 'CloneOption', depth: 0, noTags: false, reference: '', shallow: false],
                                [$class: 'SubmoduleOption', disableSubmodules: false, parentCredentials: false, recursiveSubmodules: false, reference: '', trackingSubmodules: false]
                            ],
                            userRemoteConfigs: [[
                                url: 'https://github.com/yourusername/your-repo.git'  // ← UPDATE THIS
                            ]]
                        ])
                        
                        echo "[Checkout] ✓ Repository cloned successfully"
                        echo "[Checkout] Commit Hash: ${env.GIT_COMMIT}"
                        
                        sh '''
                            echo "[Checkout] Repository contents:"
                            ls -la
                            echo ""
                            echo "[Checkout] Git log:"
                            git log -1 --pretty=format:"%h - %an - %s - %cd" --date=short
                        '''
                    } catch (Exception e) {
                        echo "[Checkout] ✗ Failed to checkout from GitHub"
                        echo "[Checkout] Error: ${e.message}"
                        throw e
                    }
                }
                echo ""
            }
        }
        
        stage('Verify Dockerfile') {
            steps {
                echo ""
                echo "═══════════════════════════════════════════════════════"
                echo "STAGE: VERIFY DOCKERFILE"
                echo "═══════════════════════════════════════════════════════"
                echo "[Verify] Checking for Dockerfile..."
                
                script {
                    if (fileExists('Dockerfile')) {
                        echo "[Verify] ✓ Dockerfile found"
                        sh '''
                            echo "[Verify] Dockerfile contents:"
                            head -20 Dockerfile
                        '''
                    } else {
                        echo "[Verify] ✗ Dockerfile NOT found in repository"
                        echo "[Verify] Please add Dockerfile to your repository root"
                        error("[Verify] Dockerfile is required")
                    }
                }
                echo ""
            }
        }
        
        stage('Build Docker Image') {
            steps {
                echo ""
                echo "═══════════════════════════════════════════════════════"
                echo "STAGE: BUILD DOCKER IMAGE"
                echo "═══════════════════════════════════════════════════════"
                echo "[Build] Starting Docker image build..."
                echo "[Build] Image Repository: ${DOCKER_IMAGE}"
                echo "[Build] Build Number Tag: ${BUILD_NUMBER}"
                echo "[Build] Git Commit: ${env.GIT_COMMIT}"
                
                script {
                    try {
                        // Build with 'latest' tag
                        dockerImage = docker.build("${DOCKER_IMAGE}:latest")
                        echo "[Build] ✓ Built ${DOCKER_IMAGE}:latest"
                        
                        // Build with build number tag
                        dockerImage = docker.build("${DOCKER_IMAGE}:${BUILD_NUMBER}")
                        echo "[Build] ✓ Built ${DOCKER_IMAGE}:${BUILD_NUMBER}"
                        
                        // Build with git commit hash
                        dockerImage = docker.build("${DOCKER_IMAGE}:${env.GIT_COMMIT.take(7)}")
                        echo "[Build] ✓ Built ${DOCKER_IMAGE}:${env.GIT_COMMIT.take(7)}"
                        
                        echo "[Build] ✓ Docker image build completed successfully"
                    } catch (Exception e) {
                        echo "[Build] ✗ Docker image build failed"
                        echo "[Build] Error: ${e.message}"
                        throw e
                    }
                }
                echo ""
            }
        }
        
        stage('Test Docker Image') {
            steps {
                echo ""
                echo "═══════════════════════════════════════════════════════"
                echo "STAGE: TEST DOCKER IMAGE"
                echo "═══════════════════════════════════════════════════════"
                echo "[Test] Testing Docker image..."
                
                script {
                    try {
                        sh '''
                            echo "[Test] Running container for testing..."
                            
                            # Run container in test mode
                            docker run -d \
                              --name test-container-${BUILD_NUMBER} \
                              -p 9090:${APP_PORT} \
                              ${DOCKER_IMAGE}:latest
                            
                            echo "[Test] Waiting for container to start..."
                            sleep 5
                            
                            echo "[Test] Container is running:"
                            docker ps | grep test-container-${BUILD_NUMBER}
                            
                            echo "[Test] ✓ Container test passed"
                            
                            # Cleanup test container
                            docker rm -f test-container-${BUILD_NUMBER}
                            echo "[Test] ✓ Test container cleaned up"
                        '''
                    } catch (Exception e) {
                        echo "[Test] ✗ Docker image test failed"
                        echo "[Test] Error: ${e.message}"
                        sh 'docker rm -f test-container-${BUILD_NUMBER} || true'
                        throw e
                    }
                }
                echo ""
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                echo ""
                echo "═══════════════════════════════════════════════════════"
                echo "STAGE: PUSH TO DOCKER HUB"
                echo "═══════════════════════════════════════════════════════"
                echo "[Push] Pushing Docker image to Docker Hub..."
                echo "[Push] Registry: ${REGISTRY_URL}"
                echo "[Push] Repository: ${DOCKER_IMAGE}"
                
                script {
                    try {
                        docker.withRegistry("${REGISTRY_URL}", "${DOCKER_CREDENTIALS_ID}") {
                            dockerImage.push('latest')
                            echo "[Push] ✓ Successfully pushed ${DOCKER_IMAGE}:latest"
                            
                            dockerImage.push("${BUILD_NUMBER}")
                            echo "[Push] ✓ Successfully pushed ${DOCKER_IMAGE}:${BUILD_NUMBER}"
                            
                            dockerImage.push("${env.GIT_COMMIT.take(7)}")
                            echo "[Push] ✓ Successfully pushed ${DOCKER_IMAGE}:${env.GIT_COMMIT.take(7)}"
                            
                            echo "[Push] ✓ All images pushed to Docker Hub"
                        }
                    } catch (Exception e) {
                        echo "[Push] ✗ Failed to push Docker image"
                        echo "[Push] Error: ${e.message}"
                        throw e
                    }
                }
                echo ""
            }
        }
        
        stage('Deploy to Remote Server') {
            when {
                branch 'main'  // Only deploy on main branch
            }
            steps {
                echo ""
                echo "═══════════════════════════════════════════════════════"
                echo "STAGE: DEPLOY TO REMOTE SERVER"
                echo "═══════════════════════════════════════════════════════"
                echo "[Deploy] Connecting to remote server..."
                echo "[Deploy] Server: ${REMOTE_USER}@${REMOTE_HOST}"
                echo "[Deploy] Container: ${CONTAINER_NAME}"
                
                sshagent(["${SSH_CREDENTIALS_ID}"]) {
                    script {
                        try {
                            sh '''
                                echo "[Deploy] Configuring SSH connection..."
                                ssh -o StrictHostKeyChecking=no -o ConnectTimeout=10 ${REMOTE_USER}@${REMOTE_HOST} << 'EOF'
                                
echo "════════════════════════════════════════════════"
echo "DEPLOYMENT EXECUTION ON REMOTE HOST"
echo "════════════════════════════════════════════════"
echo ""

echo "[1] System Information:"
echo "Date: $(date)"
echo "Hostname: $(hostname)"
echo "Docker Version: $(docker --version)"
echo ""

echo "[2] Pulling latest Docker image..."
docker pull ${DOCKER_IMAGE}:latest
echo "[✓] Image pulled successfully"
echo ""

echo "[3] Stopping and removing existing container..."
docker rm -f ${CONTAINER_NAME} || true
sleep 2
echo "[✓] Old container removed (if existed)"
echo ""

echo "[4] Starting new Docker container..."
docker run -d \
  --name ${CONTAINER_NAME} \
  -p ${CONTAINER_PORT}:${APP_PORT} \
  --restart unless-stopped \
  --health-cmd='curl -f http://localhost:${APP_PORT}/ || exit 1' \
  --health-interval=30s \
  --health-timeout=10s \
  --health-retries=3 \
  --log-driver json-file \
  --log-opt max-size=10m \
  --log-opt max-file=3 \
  ${DOCKER_IMAGE}:latest

echo "[✓] Container started successfully"
sleep 3
echo ""

echo "[5] Verifying deployment..."
echo "Running containers:"
docker ps | grep ${CONTAINER_NAME}
echo "[✓] Deployment verified"
echo ""

echo "[6] Container Details:"
docker inspect ${CONTAINER_NAME} | grep -E "State|Status|Image"
echo ""

echo "[7] Container Logs (last 20 lines):"
docker logs ${CONTAINER_NAME} | tail -20
echo ""

echo "════════════════════════════════════════════════"
echo "DEPLOYMENT COMPLETED SUCCESSFULLY"
echo "════════════════════════════════════════════════"
echo "Container Name: ${CONTAINER_NAME}"
echo "Access URL: http://${REMOTE_HOST}:${CONTAINER_PORT}"
echo "Image: ${DOCKER_IMAGE}:latest"
echo "Deployed at: $(date)"
EOF
                            '''
                            echo "[Deploy] ✓ Deployment completed successfully"
                        } catch (Exception e) {
                            echo "[Deploy] ✗ Deployment failed"
                            echo "[Deploy] Error: ${e.message}"
                            throw e
                        }
                    }
                }
                echo ""
            }
        }
    }
    
    post {
        success {
            echo ""
            echo "╔════════════════════════════════════════════════════╗"
            echo "║    ✓ GITHUB → JENKINS → DOCKER PIPELINE SUCCESS    ║"
            echo "║                                                    ║"
            echo "║  Build Number: ${BUILD_NUMBER}"
            echo "║  Git Commit: ${env.GIT_COMMIT}"
            echo "║  Docker Image: ${DOCKER_IMAGE}:latest"
            echo "║  Container: ${CONTAINER_NAME}"
            echo "║  Server: ${REMOTE_HOST}:${CONTAINER_PORT}"
            echo "║  Status: ALL STAGES COMPLETED ✓                    ║"
            echo "║                                                    ║"
            echo "║  Access your application at:                       ║"
            echo "║  http://${REMOTE_HOST}:${CONTAINER_PORT}"
            echo "║                                                    ║"
            echo "║  GitHub Repo: ${GITHUB_REPO}"
            echo "║                                                    ║"
            echo "╚════════════════════════════════════════════════════╝"
            echo ""
        }
        
        failure {
            echo ""
            echo "╔════════════════════════════════════════════════════╗"
            echo "║    ✗ GITHUB → JENKINS → DOCKER PIPELINE FAILED     ║"
            echo "║                                                    ║"
            echo "║  Build Number: ${BUILD_NUMBER}"
            echo "║  Git Commit: ${env.GIT_COMMIT}"
            echo "║  Status: FAILED - CHECK LOGS BELOW                 ║"
            echo "║                                                    ║"
            echo "║  Build URL: ${BUILD_URL}console"
            echo "║  GitHub Repo: ${GITHUB_REPO}"
            echo "║                                                    ║"
            echo "╚════════════════════════════════════════════════════╝"
            echo ""
        }
        
        always {
            echo ""
            echo "═══════════════════════════════════════════════════════"
            echo "PIPELINE SUMMARY"
            echo "═══════════════════════════════════════════════════════"
            echo "Pipeline Finished: ${new Date()}"
            echo "Total Duration: ${currentBuild.durationString}"
            echo "Build Status: ${currentBuild.result}"
            echo "Build URL: ${BUILD_URL}"
            echo "═══════════════════════════════════════════════════════"
            echo ""
        }
    }
}
