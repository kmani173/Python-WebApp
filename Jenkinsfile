pipeline {

    agent any


    environment {

        FAILURE_STAGE = "Unknown"

    }


    stages {


        stage('Checkout') {

            steps {

                script {

                    try {

                        env.FAILURE_STAGE = "Checkout"

                        checkout scm

                    }

                    catch(Exception e) {

                        error("Checkout failed: ${e.message}")

                    }

                }

            }

        }



        stage('Verify Python') {

            steps {

                script {

                    try {

                        env.FAILURE_STAGE = "Verify Python"


                        sh '''
                            python3 --version
                            pip3 --version
                        '''

                    }

                    catch(Exception e){

                        error("Python verification failed")

                    }

                }

            }

        }



        stage('Install Dependencies') {

            steps {

                script {

                    env.FAILURE_STAGE = "Install Dependencies"


                    try {

                        sh '''

                            set -o pipefail

                            pip3 install --break-system-packages \
                            -r requirements.txt 2>&1 | tee install.log

                        '''

                    }

                    catch(Exception e){

                        error("Dependency installation failed")

                    }

                }

            }

        }



        stage('Build Docker Image') {


            steps {


                script {


                    env.FAILURE_STAGE = "Build Docker Image"


                    try {


                        sh '''

                            set -o pipefail


                            docker build \
                            -t python-webapp:1.0 . \
                            2>&1 | tee docker.log


                        '''


                    }

                    catch(Exception e){


                        error("Docker image build failed")


                    }


                }


            }


        }



        stage('Run Container') {


            steps {


                script {


                    env.FAILURE_STAGE = "Run Container"


                    try {


                        sh '''

                            set -o pipefail


                            docker stop flask-demo || true

                            docker rm flask-demo || true



                            docker run -d \

                            --name flask-demo \

                            -p 5000:5000 \

                            python-webapp:1.0 \

                            2>&1 | tee run.log


                        '''


                    }

                    catch(Exception e){


                        error("Container startup failed")


                    }


                }


            }


        }





        stage('Application Test') {


            steps {


                script {


                    env.FAILURE_STAGE = "Application Test"


                    try {


                        sh '''

                            set -o pipefail


                            sleep 10


                            curl -f http://host.docker.internal:5000 \
                            2>&1 | tee app.log


                        '''


                    }

                    catch(Exception e){


                        error("Application test failed")


                    }


                }


            }


        }





        stage('AI Deployment Analysis') {


            steps {


                script {


                    sh '''


echo "===================================="

echo " AI DEPLOYMENT ANALYSIS"

echo "===================================="



cat > deployment.log <<EOF


Project:

Python-WebApp


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




if curl -s http://host.docker.internal:11434/api/tags >/dev/null

then



PROMPT=$(tr '\\n' ' ' < deployment.log)



curl -s http://host.docker.internal:11434/api/generate \

-H "Content-Type: application/json" \

-d "{

\"model\":\"smollm2:latest\",

\"prompt\":\"Analyze this successful Jenkins deployment and provide summary, improvements and recommendations. ${PROMPT}\",

\"stream\":false

}" | python3 -c "

import sys,json

print(json.load(sys.stdin)['response'])

"



else


echo "Ollama AI server is not reachable"


fi



echo "===================================="



                    '''

                }

            }

        }



    }





    post {



        success {


            echo "================================"

            echo "Deployment Successful"

            echo "================================"

        }



        failure {



            script {



                echo "================================"

                echo " AI ROOT CAUSE ANALYSIS"

                echo "================================"



                def log = ""



                if(fileExists('install.log')){


                    log = readFile('install.log')


                }

                else if(fileExists('docker.log')){


                    log = readFile('docker.log')


                }

                else if(fileExists('run.log')){


                    log = readFile('run.log')


                }

                else if(fileExists('app.log')){


                    log = readFile('app.log')


                }

                else {


                    log = currentBuild.rawBuild
                    .getLog(300)
                    .join("\n")


                }





                writeFile(

                    file:"failure.log",

                    text:log

                )





                sh """


if curl -s http://host.docker.internal:11434/api/tags >/dev/null

then



PROMPT=\$(tr '\\n' ' ' < failure.log)



curl -s http://host.docker.internal:11434/api/generate \\

-H "Content-Type: application/json" \\

-d "{

\\"model\\":\\"smollm2:latest\\",

\\"prompt\\":

\\"You are a Senior DevOps Engineer.

Analyze Jenkins pipeline failure.

Failed Stage:

${FAILURE_STAGE}


Logs:

\$PROMPT


Return:

1. Build Status

2. Failed Stage

3. Root Cause

4. Exact Error

5. Why it Happened

6. Recommended Fix

7. Severity

8. Confidence Score

\\",

\\"stream\\":false

}" | python3 -c "

import sys,json

print(json.load(sys.stdin)['response'])

"



else


echo "Ollama AI server not reachable"


fi



echo "================================"

echo "Deployment Failed"

echo "================================"



"""


            }


        }


    }


}
