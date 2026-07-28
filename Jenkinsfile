pipeline {
    agent any

    environment {
        FAILURE_STAGE = ""
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
                echo "======================================="
                echo "Deployment Successful"
                echo "======================================="
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

                def failureLog = ""

                if (fileExists("install.log")) {
                    failureLog = readFile("install.log")
                } else if (fileExists("docker.log")) {
                    failureLog = readFile("docker.log")
                } else if (fileExists("run.log")) {
                    failureLog = readFile("run.log")
                } else if (fileExists("app.log")) {
                    failureLog = readFile("app.log")
                } else {
                    failureLog = "No log file found."
                }

                writeFile file: "prompt.txt", text: """
You are a Senior DevOps Engineer.

Analyze this failed Jenkins pipeline.

BUILD STATUS:
FAILED

FAILED STAGE:
${env.FAILURE_STAGE}

CONSOLE LOG:
${failureLog}

Provide your answer exactly in this format.

====================================

BUILD STATUS

FAILED

FAILED STAGE

ROOT CAUSE

EXACT ERROR

WHY IT HAPPENED

RECOMMENDED FIX

SEVERITY

CONFIDENCE SCORE

====================================
"""

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
