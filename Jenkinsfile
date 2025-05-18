pipeline {
    agent any
    
    environment {
        DOCKER_COMPOSE_VERSION = '2.17.2'
        APP_NAME = 'book-management'
        DOCKER_REGISTRY = 'your-registry-url' // Replace with your Docker registry if using one
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Install Dependencies') {
            steps {
                // Install Docker Compose if needed
                sh '''
                if ! command -v docker-compose &> /dev/null; then
                    echo "Installing Docker Compose..."
                    curl -L "https://github.com/docker/compose/releases/download/${DOCKER_COMPOSE_VERSION}/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
                    chmod +x /usr/local/bin/docker-compose
                fi
                '''
            }
        }
        
        stage('Build Frontend') {
            steps {
                dir('frontend') {
                    sh 'docker build -t ${APP_NAME}-frontend:latest .'
                }
            }
        }
        
        stage('Build Backend') {
            steps {
                dir('backend') {
                    sh 'docker build -t ${APP_NAME}-backend:latest .'
                }
            }
        }
        
        stage('Run Tests') {
            parallel {
                stage('Frontend Tests') {
                    steps {
                        dir('frontend') {
                            sh 'echo "Running frontend tests..."'
                            // Add actual test commands for your frontend
                            // Example: sh 'docker run --rm ${APP_NAME}-frontend:latest npm test'
                        }
                    }
                }
                
                stage('Backend Tests') {
                    steps {
                        dir('backend') {
                            sh 'echo "Running backend tests..."'
                            // Add actual test commands for your backend
                            // Example: sh 'docker run --rm ${APP_NAME}-backend:latest npm test'
                        }
                    }
                }
            }
        }
        
        stage('Push Images') {
            when {
                branch 'main' // Only push images when building the main branch
            }
            steps {
                withCredentials([string(credentialsId: 'docker-registry-credentials', variable: 'DOCKER_CREDS')]) {
                    sh '''
                    echo $DOCKER_CREDS | docker login ${DOCKER_REGISTRY} -u username --password-stdin
                    
                    # Tag and push the frontend image
                    docker tag ${APP_NAME}-frontend:latest ${DOCKER_REGISTRY}/${APP_NAME}-frontend:latest
                    docker tag ${APP_NAME}-frontend:latest ${DOCKER_REGISTRY}/${APP_NAME}-frontend:${BUILD_NUMBER}
                    docker push ${DOCKER_REGISTRY}/${APP_NAME}-frontend:latest
                    docker push ${DOCKER_REGISTRY}/${APP_NAME}-frontend:${BUILD_NUMBER}
                    
                    # Tag and push the backend image
                    docker tag ${APP_NAME}-backend:latest ${DOCKER_REGISTRY}/${APP_NAME}-backend:latest
                    docker tag ${APP_NAME}-backend:latest ${DOCKER_REGISTRY}/${APP_NAME}-backend:${BUILD_NUMBER}
                    docker push ${DOCKER_REGISTRY}/${APP_NAME}-backend:latest
                    docker push ${DOCKER_REGISTRY}/${APP_NAME}-backend:${BUILD_NUMBER}
                    '''
                }
            }
        }
        
        stage('Deploy') {
            when {
                branch 'main' // Only deploy when building the main branch
            }
            steps {
                // Make sure .env file exists for backend
                sh '''
                if [ ! -f ./backend/.env ]; then
                    echo "Creating sample .env file for backend"
                    echo "PORT=5555" > ./backend/.env
                    echo "DB_URI=mongodb://localhost:27017/book-management" >> ./backend/.env
                    # Add other environment variables as needed
                fi
                '''
                
                // Deploy using docker-compose
                sh 'docker-compose down || true'
                sh 'docker-compose up -d'
                
                // Verify deployment
                sh '''
                echo "Verifying deployment..."
                sleep 10
                if docker ps | grep -q book-management-backend && docker ps | grep -q book-management-frontend; then
                    echo "Deployment successful!"
                else
                    echo "Deployment failed!"
                    exit 1
                fi
                '''
            }
        }
    }
    
    post {
        always {
            // Clean up unused Docker resources
            sh 'docker system prune -f'
            
            // Archive artifacts and test results if available
            archiveArtifacts artifacts: '**/target/*.jar', allowEmptyArchive: true
            junit '**/target/surefire-reports/*.xml', allowEmptyResults: true
        }
        
        success {
            echo 'Build and deployment successful!'
        }
        
        failure {
            echo 'Build or deployment failed!'
        }
    }
}
