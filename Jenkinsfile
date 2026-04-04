pipeline {
    agent any

    stages {
        stage('Build Docker Image') {
            steps {
                bat 'docker build -t civicsense-app .'
            }
        }

        stage('Run Container') {
            steps {
                bat 'docker run -d -p 3000:3000 civicsense-app'
            }
        }
    }
}
