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
                sh '''
                python3 -m venv venv
                . venv/bin/activate
                pip install --upgrade pip
                pip install -r requirements.txt
                '''
            }
        }

        stage('Run Application Test') {
            steps {
                sh '''
                . venv/bin/activate
                python app.py &
                sleep 10
                curl http://localhost:5000
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
