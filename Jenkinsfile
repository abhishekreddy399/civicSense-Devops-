pipeline {
    agent any

    environment {
        // REQUIRED: Set these in Jenkins Credentials (Manage Jenkins > Credentials)
        MONGO_URI = credentials('mongo-uri')
        JWT_SECRET = credentials('jwt-secret')
        
        // EC2 Configuration
        EC2_USER = "ubuntu" // or 'ec2-user' if using Amazon Linux
        EC2_IP = "YOUR_EC2_PUBLIC_IP" 
        SSH_CRED_ID = "ec2-ssh-key"
        
        // App Configuration
        APP_NAME = "civic-sense"
        DEPLOY_PATH = "/home/${EC2_USER}/${APP_NAME}"
    }

    triggers {
        // Poll GitHub every 5 minutes for changes (Perfect for Local Jenkins)
        pollSCM('H/5 * * * *')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Prepare Environment') {
            steps {
                echo 'Creating .env file for backend...'
                // Create local .env for testing/building if needed
                writeFile file: 'backend/.env', text: """
MONGO_URI=${MONGO_URI}
JWT_SECRET=${JWT_SECRET}
PORT=5000
NODE_ENV=production
"""
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent([SSH_CRED_ID]) {
                    echo "Deploying to EC2 (${EC2_IP})..."
                    
                    // 1. Ensure directory exists
                    sh "ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_IP} 'mkdir -p ${DEPLOY_PATH}'"
                    
                    // 2. Sync files to EC2 (Excluding node_modules)
                    sh """
                        tar --exclude='node_modules' --exclude='.git' -czf - . | ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_IP} "cd ${DEPLOY_PATH} && tar -xzf -"
                    """
                    
                    // 3. Start containers on EC2
                    sh """
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_IP} '
                            cd ${DEPLOY_PATH}
                            export MONGO_URI="${MONGO_URI}"
                            export JWT_SECRET="${JWT_SECRET}"
                            export REACT_APP_API_URL="http://${EC2_IP}:5000"
                            export CLIENT_URL="http://${EC2_IP}"
                            docker-compose down
                            docker-compose up -d --build
                        '
                    """

                }
            }
        }

        stage('Health Check') {
            steps {
                echo 'Verifying deployment...'
                // Wait a bit for containers to start
                sleep 10
                sh "curl -f http://${EC2_IP}:5000/api/health || echo 'Health check failed, check logs'"
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished.'
        }
        success {
            echo 'Deployment successful! 🎉'
        }
        failure {
            echo 'Deployment failed. Check Jenkins logs.'
        }
    }
}

