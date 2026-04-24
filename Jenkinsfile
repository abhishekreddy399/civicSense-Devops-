pipeline {
    agent any

    environment {
        // These should be set in Jenkins Credentials (Manage Jenkins > Credentials)
        MONGO_URI = credentials('mongo-uri')
        JWT_SECRET = credentials('jwt-secret')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Prepare') {
            steps {
                echo 'Building Docker Images...'
                bat 'docker-compose build'
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying containers...'
                // -d starts in detached mode, --remove-orphans cleans up old containers
                bat 'docker-compose up -d --remove-orphans'
            }
        }

        stage('Health Check') {
            steps {
                echo 'Verifying deployment...'
                bat 'curl -f http://localhost:5000/api/health || exit 1'
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished.'
        }
        failure {
            echo 'Deployment failed. Check Docker logs.'
        }
    }
}
