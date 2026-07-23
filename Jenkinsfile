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
                    pip3 install --break-system-packages -r requirements.txt
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t python-webapp:1.0 .
                '''
            }
        }

        stage('Run Container') {
            steps {
                sh '''
                    docker stop flask-demo || true
                    docker rm flask-demo || true

                    docker run -d \
                      --name flask-demo \
                      -p 5000:5000 \
                      python-webapp:1.0
                '''
            }
        }

        stage('Application Test') {
            steps {
                sh '''
                    sleep 10
                    curl http://host.docker.internal:5000
                '''
            }
        }
    }

    post {
        success {
            echo 'Python Application Deployment Successful'
        }
        failure {
            echo 'Deployment Failed'
        }
    }
}
