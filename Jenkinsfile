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
                    echo "Installing Python dependencies..."
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

        stage('AI Deployment Analysis') {
            when {
                expression { currentBuild.currentResult == 'SUCCESS' }
            }
            steps {
                sh '''
                echo "Deployment Successful."
                '''
            }
        }
    }

    post {

        success {
            echo "Python Application Deployment Successful"
        }

        failure {

            script {

                echo "Collecting Jenkins Console Log..."

                def log = currentBuild.rawBuild.getLog(3000).join("\n")

                writeFile file: 'jenkins.log', text: log

                sh '''
                echo ""
                echo "==========================================="
                echo " AI ROOT CAUSE ANALYSIS"
                echo "==========================================="
                echo ""

                LOG=$(tr '\n' ' ' < jenkins.log | sed 's/"/\\\\\\"/g')

                curl -X POST http://host.docker.internal:11434/api/generate \
                  -H "Content-Type: application/json" \
                  -d "{\\"model\\":\\"smollm2:latest\\",\\"prompt\\":\\"You are a Senior DevOps Engineer.

Analyze this failed Jenkins pipeline.

Provide your response in the following format:

====================================
BUILD STATUS
FAILED

FAILED STAGE

ROOT CAUSE

EXACT ERROR

WHY IT HAPPENED

RECOMMENDED FIX

SEVERITY

CONFIDENCE
====================================

Jenkins Console Log:

$LOG\\",\\"stream\\":false}"

                echo ""
                echo "==========================================="
                echo " AI RCA Completed"
                echo "==========================================="
                '''
            }

            echo "Deployment Failed"
        }
    }
}
