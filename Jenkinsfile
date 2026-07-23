pipeline {

    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Verify Docker') {
            steps {
                sh 'docker --version'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t python-webapp .
                '''
            }
        }

        stage('Remove Old Container') {
            steps {
                sh 'python3 --version'
            }
        }

        stage('Run Flask Container') {
            steps {
                sh 'pip3 install -r requirements.txt'
            }
        }

        stage('Run Application Test') {
            steps {
                sh '''
                python3 app.py &
                sleep 5
                curl http://127.0.0.1:5000
                pkill -f app.py || true
                '''
            }
        }

    }

    post {

        success {
            echo "Deployment Successful"
        }

        failure {
            echo "Deployment Failed"
        }

    }

}
