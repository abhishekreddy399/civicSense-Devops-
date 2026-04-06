pipeline {
    agent any

    environment {
        MONGO_URI = credentials('mongo-uri')
    }

    stages {
        stage('Build Docker Image') {
            steps {
                bat 'docker build -t civicsense-app ./backend'
            }
        }

        stage('Stop Old Container') {
            steps {
                bat 'docker stop civicsense || exit 0'
                bat 'docker rm civicsense || exit 0'
            }
        }

       stage('Run Container') {
    steps {
        bat '''
        docker run -d -p 5000:5000 ^
        -e MONGO_URI="%MONGO_URI%" ^
        --name civicsense civicsense-app
        '''
    }
}
    }
}
