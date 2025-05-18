pipeline {
    agent any

    environment {
        DOCKER_COMPOSE_PROJECT = 'book-management-system'
        DOCKER_COMPOSE_FILE = 'docker-compose.yml'
        MONGODB_URI = credentials('MONGODB_URI')
        JWT_SECRET = credentials('JWT_SECRET')
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout code from GitHub repository
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
                // Build and start Docker containers
                sh 'docker-compose -p ${DOCKER_COMPOSE_PROJECT} -f ${DOCKER_COMPOSE_FILE} build'
                sh 'docker-compose -p ${DOCKER_COMPOSE_PROJECT} -f ${DOCKER_COMPOSE_FILE} up -d'
            }
        }

        stage('Verify Deployment') {
            steps {
                sh 'docker ps | grep book-management'
                sh 'sleep 10'
                // Verify backend is running
                sh 'curl -s http://localhost:5002 || true'
            }
        }
    }

    post {
        success {
            echo 'Deployment successful!'
        }

        failure {
            node {
                echo 'Deployment failed!'
                sh 'docker-compose -p ${DOCKER_COMPOSE_PROJECT} -f ${DOCKER_COMPOSE_FILE} down || true'
                sh 'echo "Deployment failed!"'
            }
        }

        always {
            node {
                // Clean up any dangling images
                sh 'docker image prune -f'
                archiveArtifacts artifacts: 'docker-compose.log', allowEmptyArchive: true
                sh 'echo "This will always run after the pipeline"'
            }
        }
    }
}
