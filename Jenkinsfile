pipeline {
    agent any

    environment {
        FAILURE_STAGE = ""
        OLLAMA_URL = "http://host.docker.internal:11434/api/generate"
        MODEL = "smollm2:latest"
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

                    env.FAILURE_STAGE = "Install Dependencies"

                    def status = sh(
                        script: '''
                            echo "Installing dependencies..."
                            pip3 install --break-system-packages -r requirements.txt \
                            > install.log 2>&1
                        ''',
                        returnStatus: true
                    )

                    if (status != 0) {
                        error("Dependency Installation Failed")
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {

                    env.FAILURE_STAGE = "Build Docker Image"

                    def status = sh(
                        script: '''
                            echo "Building Docker image..."
                            docker build -t python-webapp:1.0 . \
                            > docker.log 2>&1
                        ''',
                        returnStatus: true
                    )

                    if (status != 0) {
                        error("Docker Build Failed")
                    }
                }
            }
        }

        stage('Run Container') {
            steps {
                script {

                    env.FAILURE_STAGE = "Run Container"

                    def status = sh(
                        script: '''
                            docker stop flask-demo || true
                            docker rm flask-demo || true

                            docker run -d \
                                --name flask-demo \
                                -p 5000:5000 \
                                python-webapp:1.0 \
                                > run.log 2>&1
                        ''',
                        returnStatus: true
                    )

                    if (status != 0) {
                        error("Container Failed")
                    }
                }
            }
        }

        stage('Application Test') {
            steps {
                script {

                    env.FAILURE_STAGE = "Application Test"

                    def status = sh(
                        script: '''
                            sleep 10

                            curl -v http://host.docker.internal:5000 \
                            > app.log 2>&1
                        ''',
                        returnStatus: true
                    )

                    if (status != 0) {
                        error("Application Test Failed")
                    }
                }
            }
        }

        stage('Deployment Successful') {
            steps {
                echo "Application deployed successfully."
            }
        }

    }

    post {

        success {
            echo "========================================"
            echo "BUILD SUCCESS"
            echo "========================================"
        }

        failure {

            script {

                def failureLog = ""

                if (env.FAILURE_STAGE == "Install Dependencies" && fileExists("install.log")) {
                    failureLog = readFile("install.log")
                }
                else if (env.FAILURE_STAGE == "Build Docker Image" && fileExists("docker.log")) {
                    failureLog = readFile("docker.log")
                }
                else if (env.FAILURE_STAGE == "Run Container" && fileExists("run.log")) {
                    failureLog = readFile("run.log")
                }
                else if (env.FAILURE_STAGE == "Application Test" && fileExists("app.log")) {
                    failureLog = readFile("app.log")
                }
                else {
                    failureLog = "No log file found."
                }

                writeFile file: "prompt.txt", text: """
You are a Senior DevOps Engineer.

Analyze the failed Jenkins pipeline.

Failed Stage:
${env.FAILURE_STAGE}

Console Log:

${failureLog}

Return ONLY the report below.

BUILD STATUS
FAILED

FAILED STAGE

ROOT CAUSE

EXACT ERROR

WHY IT HAPPENED

RECOMMENDED FIX

SEVERITY

CONFIDENCE SCORE

Do not return JSON.
Do not use Markdown.
Only return the report.
"""
            }

            sh '''
                echo ""
                echo "========================================"
                echo "AI ROOT CAUSE ANALYSIS"
                echo "========================================"

                PROMPT=$(jq -Rs . < prompt.txt)

                RESPONSE=$(curl -s $OLLAMA_URL \
                -H "Content-Type: application/json" \
                -d "{
                    \\"model\\": \\"smollm2:latest\\",
                    \\"prompt\\": $PROMPT,
                    \\"stream\\": false
                }")

                echo ""
                echo "========================================"
                echo "AI RCA REPORT"
                echo "========================================"

                echo "$RESPONSE" | jq -r '.response'

                echo ""
                echo "========================================"
                echo "AI RCA COMPLETED"
                echo "========================================"
            '''

            echo "Pipeline Failed."
        }

        always {

            archiveArtifacts artifacts: '*.log', allowEmptyArchive: true

            cleanWs(
                deleteDirs: true,
                disableDeferredWipeout: true
            )
        }
    }
}
