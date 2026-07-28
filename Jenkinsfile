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

                    sh '''
                        set -o pipefail
                        echo "Installing dependencies..."

                        pip3 install --break-system-packages -r requirements.txt \
                        2>&1 | tee install.log
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {

                    env.FAILURE_STAGE = "Build Docker Image"

                    sh '''
                        set -o pipefail

                        docker build -t python-webapp:1.0 . \
                        2>&1 | tee docker.log
                    '''
                }
            }
        }

        stage('Run Container') {
            steps {
                script {

                    env.FAILURE_STAGE = "Run Container"

                    sh '''
                        docker stop flask-demo || true
                        docker rm flask-demo || true

                        set -o pipefail

                        docker run -d \
                          --name flask-demo \
                          -p 5000:5000 \
                          python-webapp:1.0 \
                          2>&1 | tee run.log
                    '''
                }
            }
        }

        stage('Application Test') {
            steps {
                script {

                    env.FAILURE_STAGE = "Application Test"

                    sh '''
                        sleep 10

                        set -o pipefail

                        curl -v http://host.docker.internal:5000 \
                        2>&1 | tee app.log
                    '''
                }
            }
        }

    }

    post {

        success {

            echo "========================================"
            echo "Application deployed successfully."
            echo "========================================"
        }

        failure {

            script {

                String failureLog = ""

                switch(env.FAILURE_STAGE) {

                    case "Install Dependencies":
                        failureLog = fileExists("install.log") ? readFile("install.log") : ""
                        break

                    case "Build Docker Image":
                        failureLog = fileExists("docker.log") ? readFile("docker.log") : ""
                        break

                    case "Run Container":
                        failureLog = fileExists("run.log") ? readFile("run.log") : ""
                        break

                    case "Application Test":
                        failureLog = fileExists("app.log") ? readFile("app.log") : ""
                        break

                    default:
                        failureLog = currentBuild.rawBuild.getLog(300).join("\n")
                }

                writeFile file: "prompt.txt", text: """
You are a Senior DevOps Engineer.

Analyze this failed Jenkins pipeline.

Failed Stage:
${env.FAILURE_STAGE}

Console Log:

${failureLog}

Return ONLY this format.

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
Do not add markdown.
Only return the report.
"""

            }

            sh '''
                echo ""
                echo "====================================="
                echo " AI ROOT CAUSE ANALYSIS"
                echo "====================================="

                PROMPT=$(jq -Rs . < prompt.txt)

                curl -s $OLLAMA_URL \
                -H "Content-Type: application/json" \
                -d "{
                    \\"model\\":\\"$MODEL\\",
                    \\"prompt\\":$PROMPT,
                    \\"stream\\":false
                }" \
                | jq -r '.response'

                echo ""
                echo "====================================="
                echo " AI RCA COMPLETED"
                echo "====================================="
            '''

            echo "Deployment Failed"
        }
    }
}
