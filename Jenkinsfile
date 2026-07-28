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
                    env.FAILURE_STAGE = "Install Dependencies"

                    int status = sh(
                        script: '''
                            set -o pipefail
                            pip3 install --break-system-packages -r requirements.txt 2>&1 | tee install.log
                        ''',
                        returnStatus: true
                    )

                    if (status != 0) {
                        error("Dependency installation failed")
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    env.FAILURE_STAGE = "Build Docker Image"

                    sh '''
                        set -o pipefail
                        docker build -t python-webapp:1.0 . 2>&1 | tee docker.log
                    '''
                }
            }
        }

        stage('Run Container') {
            steps {
                script {
                    env.FAILURE_STAGE = "Run Container"

                    sh '''
                        set -o pipefail

                        docker stop flask-demo || true
                        docker rm flask-demo || true

                        docker run -d \
                          --name flask-demo \
                          -p 5000:5000 \
                          python-webapp:1.0 2>&1 | tee run.log
                    '''
                }
            }
        }

        stage('Application Test') {
            steps {
                script {
                    env.FAILURE_STAGE = "Application Test"

                    sh '''
                        set -o pipefail

                        sleep 10

                        curl http://host.docker.internal:5000 2>&1 | tee app.log
                    '''
                }
            }
        }

        stage('AI Deployment Analysis') {

            when {
                expression {
                    currentBuild.currentResult == null ||
                    currentBuild.currentResult == 'SUCCESS'
                }
            }

            steps {

                sh '''

echo "======================================"
echo " AI DEPLOYMENT ANALYSIS"
echo "======================================"

cat > deployment.log <<EOF
Project Name: Python-WebApp

Python Version:
$(python3 --version)

Docker Image:
python-webapp:1.0

Container:
flask-demo

Deployment Status:
SUCCESS

Application Test:
PASSED
EOF

PROMPT=$(tr '\\n' ' ' < deployment.log | sed 's/"/\\\\\\"/g')

curl -s http://host.docker.internal:11434/api/generate \
-H "Content-Type: application/json" \
-d "{\\"model\\":\\"smollm2:latest\\",\\"prompt\\":\\"Analyze this deployment and provide Deployment Summary, Issues and Recommendations. $PROMPT\\",\\"stream\\":false}" \
| python3 -c "import sys,json;print(json.load(sys.stdin)['response'])"

echo "======================================"

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

                def log = ""

                if (fileExists("install.log")) {
                    log = readFile("install.log")
                }
                else if (fileExists("docker.log")) {
                    log = readFile("docker.log")
                }
                else if (fileExists("run.log")) {
                    log = readFile("run.log")
                }
                else if (fileExists("app.log")) {
                    log = readFile("app.log")
                }
                else {
                    log = currentBuild.rawBuild.getLog(300).join("\n")
                }

                writeFile file: "failure.log", text: log
            }

            sh '''

echo "======================================"
echo " AI ROOT CAUSE ANALYSIS"
echo "======================================"

PROMPT=$(tr '\\n' ' ' < failure.log | sed 's/"/\\\\\\"/g')

curl -s http://host.docker.internal:11434/api/generate \
-H "Content-Type: application/json" \
-d "{\\"model\\":\\"smollm2:latest\\",\\"prompt\\":\\"You are a Senior DevOps Engineer. Analyze the following failed Jenkins pipeline. Failed Stage: ${FAILURE_STAGE}. Console Log: $PROMPT. Return Build Status, Failed Stage, Root Cause, Exact Error, Why it Happened, Recommended Fix, Severity and Confidence Score.\\",\\"stream\\":false}" \
| python3 -c "import sys,json;print(json.load(sys.stdin)['response'])"

echo "======================================"

'''

            echo "Deployment Failed"
        }

    }
}
