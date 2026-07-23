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
                    python3 -m pip install --upgrade pip
                    pip3 install -r requirements.txt
                '''
            }
        }

        stage('Run Application') {
            steps {
                sh '''
                    timeout 10 python3 app.py || true
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                    python3 app.py &
                    sleep 5
                    curl http://localhost:5000/health
                    pkill -f app.py
                '''
            }
        }

    }

    post {

        success {
            echo 'Python Web Application executed successfully.'
        }

        failure {
            echo 'Python Web Application failed.'
        }

    }

}