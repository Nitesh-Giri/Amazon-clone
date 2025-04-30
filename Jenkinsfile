pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'amazon-clone'
        DOCKER_TAG = "${BUILD_NUMBER}"
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
                // Stop and remove existing container
                sh "docker-compose down || true"
                
                // Start new container
                sh "docker-compose up -d"
            }
        }
    }
    
    post {
        always {
            // Clean up
            sh "docker system prune -f"
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
} 