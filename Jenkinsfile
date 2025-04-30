pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'amazon-clone'
        DOCKER_TAG = "${BUILD_NUMBER}"
        CONTAINER_NAME = "amazon-clone-${BUILD_NUMBER}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                // Clean workspace and checkout code
                cleanWs()
                checkout scm
            }
        }
        
        stage('Build Docker Image') {
            steps {
                // Build Docker image
                sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
            }
        }
        
        stage('Test') {
            steps {
                // Run basic tests
                sh '''
                    # Check if index.html exists
                    if [ ! -f "public/index.html" ]; then
                        echo "index.html not found!"
                        exit 1
                    fi
                    
                    # Check if style.css exists
                    if [ ! -f "public/style.css" ]; then
                        echo "style.css not found!"
                        exit 1
                    fi
                    
                    # Check if script.js exists
                    if [ ! -f "public/script.js" ]; then
                        echo "script.js not found!"
                        exit 1
                    fi
                '''
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    // Stop and remove existing containers
                    sh '''
                        # Stop any running containers with the same name
                        docker stop amazon-clone || true
                        docker rm amazon-clone || true
                        
                        # Remove old containers
                        docker-compose down --remove-orphans || true
                        
                        # Start new container with unique name
                        CONTAINER_NAME=${CONTAINER_NAME} docker-compose up -d
                    '''
                }
            }
        }
    }
    
    post {
        always {
            // Clean up
            sh '''
                # Remove old containers and images
                docker system prune -f
                
                # Remove old containers with the same name pattern
                docker ps -a | grep "amazon-clone-" | grep -v "${CONTAINER_NAME}" | awk '{print $1}' | xargs -r docker rm -f
            '''
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
            // Clean up on failure
            sh '''
                docker-compose down --remove-orphans || true
                docker system prune -f
            '''
        }
    }
} 