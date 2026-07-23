pipeline {

    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Verify Python') {
            steps {
                sh 'python3 --version'
            }
        }

        stage('Install Dependencies') {
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
            echo 'Python Web Application executed successfully.'
        }

        failure {
            echo 'Application failed.'
        }
    }
}