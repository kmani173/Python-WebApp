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
                sh '''
                python3 --version
                pip3 --version
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                pip3 install --break-system-packages --upgrade pip
                pip3 install --break-system-packages -r requirements.txt
                '''
            }
        }

        stage('Run Flask Application') {
            steps {
                sh '''
                python3 app.py &
                sleep 10
                curl http://127.0.0.1:5000
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
