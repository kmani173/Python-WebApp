pipeline {
    agent any

    environment {
        FAILURE_STAGE = ""
        FAILURE_LOG = ""
    }

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
                script {

                    def status = sh(
                        script: '''
                        echo "Installing Python dependencies..."
                        pip3 install --break-system-packages -r requirements.txt > install.log 2>&1
                        ''',
                        returnStatus: true
                    )

                    if (status != 0) {
                        env.FAILURE_STAGE = "Install Dependencies"
                        env.FAILURE_LOG = readFile('install.log')
                        error("Dependency Installation Failed")
                    }

                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {

                    def status = sh(
                        script: '''
                        docker build -t python-webapp:1.0 . > docker.log 2>&1
                        ''',
                        returnStatus: true
                    )

                    if (status != 0) {
                        env.FAILURE_STAGE = "Build Docker Image"
                        env.FAILURE_LOG = readFile('docker.log')
                        error("Docker Build Failed")
                    }

                }
            }
        }

        stage('Run Container') {
            steps {
                script {

                    def status = sh(
                        script: '''
                        docker stop flask-demo || true
                        docker rm flask-demo || true

                        docker run -d \
                        --name flask-demo \
                        -p 5000:5000 \
                        python-webapp:1.0 > run.log 2>&1
                        ''',
                        returnStatus: true
                    )

                    if (status != 0) {
                        env.FAILURE_STAGE = "Run Container"
                        env.FAILURE_LOG = readFile('run.log')
                        error("Container Failed")
                    }

                }
            }
        }

        stage('Application Test') {
            steps {
                script {

                    def status = sh(
                        script: '''
                        sleep 10
                        curl http://host.docker.internal:5000 > app.log 2>&1
                        ''',
                        returnStatus: true
                    )

                    if (status != 0) {
                        env.FAILURE_STAGE = "Application Test"
                        env.FAILURE_LOG = readFile('app.log')
                        error("Application Test Failed")
                    }

                }
            }
        }

        stage('AI Deployment Analysis') {
            when {
                expression {
                    currentBuild.currentResult == 'SUCCESS'
                }
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

                writeFile file: "failure.log", text: env.FAILURE_LOG

                def prompt = """
You are a Senior DevOps Engineer.

Analyze this failed Jenkins pipeline.

Build Status:
FAILED

Failed Stage:
${env.FAILURE_STAGE}

Console Log:
${env.FAILURE_LOG}

Provide:

1. Root Cause
2. Exact Error
3. Why It Happened
4. Recommended Fix
5. Severity
6. Confidence Score
"""

                writeFile file: "prompt.txt", text: prompt

                sh '''
                echo ""
                echo "========================================"
                echo " AI ROOT CAUSE ANALYSIS"
                echo "========================================"

                PROMPT=$(tr '\n' ' ' < prompt.txt | sed 's/"/\\\\\\"/g')

                curl -X POST http://host.docker.internal:11434/api/generate \
                  -H "Content-Type: application/json" \
                  -d "{\\"model\\":\\"smollm2:latest\\",\\"prompt\\":\\"$PROMPT\\",\\"stream\\":false}"

                echo ""
                echo "========================================"
                echo " AI RCA COMPLETED"
                echo "========================================"
                '''
            }

            echo "Deployment Failed"

        }

    }

}
