pipeline {
    agent any
    
    environment {
        DOCKER_COMPOSE_PROJECT = 'book-management-system'
        DOCKER_COMPOSE_FILE = 'docker-compose.yml'
        MONGODB_URI = credentials('MONGODB_URI')
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Setup Environment') {
            steps {
                sh '''
                    cat > backend/.env << EOF
NODE_ENV=production
PORT=5555
MONGODB_URI=${MONGODB_URI}
EOF
                '''
            }
        }
        
        stage('Build and Start Containers') {
            steps {
                sh 'docker-compose -p ${DOCKER_COMPOSE_PROJECT} -f ${DOCKER_COMPOSE_FILE} build'
                sh 'docker-compose -p ${DOCKER_COMPOSE_PROJECT} -f ${DOCKER_COMPOSE_FILE} up -d'
                // Optionally capture logs for artifact archiving
                sh 'docker-compose -p ${DOCKER_COMPOSE_PROJECT} -f ${DOCKER_COMPOSE_FILE} logs > docker-compose.log || true'
            }
        }
        
        stage('Verify Deployment') {
            steps {
                sh 'docker ps | grep book-management'
                sh 'sleep 10'
                sh 'curl -s http://localhost:5002 || true'
            }
        }
    }
    
    post {
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Deployment failed!'
            sh 'docker-compose -p ${DOCKER_COMPOSE_PROJECT} -f ${DOCKER_COMPOSE_FILE} down || true'
        }
        always {
            // Create an empty log file if it doesn't exist to avoid archiving errors
            sh 'touch docker-compose.log || true'
            archiveArtifacts artifacts: 'docker-compose.log', allowEmptyArchive: true
        }
    }
}
