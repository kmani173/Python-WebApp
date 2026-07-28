pipeline {

    agent any


    environment {

        FAILURE_STAGE = "Unknown"

    }


    stages {


        stage('Verify Python') {

            steps {

                script {

                    env.FAILURE_STAGE = "Verify Python"

                    sh '''
                    /usr/bin/bash -c "

                    python3 --version

                    pip3 --version

                    "

                    '''

                }

            }

        }



        stage('Install Dependencies') {

            steps {

                script {

                    env.FAILURE_STAGE = "Install Dependencies"


                    sh '''
                    /usr/bin/bash -c "

                    set -o pipefail


                    pip3 install --break-system-packages \
                    -r requirements.txt 2>&1 | tee install.log


                    "

                    '''

                }

            }

        }





        stage('Build Docker Image') {


            steps {


                script {


                    env.FAILURE_STAGE = "Build Docker Image"



                    sh '''
                    /usr/bin/bash -c "

                    set -o pipefail


                    docker build \
                    -t python-webapp:1.0 . \
                    2>&1 | tee docker.log


                    "

                    '''


                }


            }


        }





        stage('Run Container') {


            steps {


                script {


                    env.FAILURE_STAGE = "Run Container"



                    sh '''
                    /usr/bin/bash -c "

                    set -o pipefail


                    docker stop flask-demo || true


                    docker rm flask-demo || true



                    docker run -d \
                    --name flask-demo \
                    -p 5000:5000 \
                    python-webapp:1.0 \
                    2>&1 | tee run.log


                    "

                    '''


                }


            }


        }





        stage('Application Test') {


            steps {


                script {


                    env.FAILURE_STAGE = "Application Test"



                    sh '''
                    /usr/bin/bash -c "

                    set -o pipefail


                    sleep 10



                    curl -f http://host.docker.internal:5000 \
                    2>&1 | tee app.log



                    "

                    '''


                }


            }


        }





        stage('AI Deployment Analysis') {


            steps {


                script {


                    sh '''
                    /usr/bin/bash -c "


                    echo '===================================='

                    echo 'AI DEPLOYMENT ANALYSIS'

                    echo '===================================='



                    cat > deployment.log <<EOF

Project: Python-WebApp

Docker Image: python-webapp:1.0

Container: flask-demo

Status: SUCCESS

Application Test: PASSED

EOF



                    PROMPT=\$(tr '\\n' ' ' < deployment.log)



                    curl -s --max-time 120 \
                    http://host.docker.internal:11434/api/generate \
                    -H 'Content-Type: application/json' \
                    -d \"{

                    \\\"model\\\":\\\"smollm2:latest\\\",

                    \\\"prompt\\\":\\\"Analyze this successful deployment and provide summary and recommendations. \$PROMPT\\\",

                    \\\"stream\\\":false

                    }\" | python3 -c \"

                    import sys,json

                    data=json.load(sys.stdin)

                    print(data.get('response',data))

                    "



                    echo '===================================='


                    "

                    '''

                }

            }

        }


    }





    post {


        success {


            echo "===================================="

            echo "Python Application Deployment Successful"

            echo "===================================="


        }





        failure {


            script {



                echo "===================================="

                echo "AI ROOT CAUSE ANALYSIS"

                echo "===================================="



                def log = ""



                if(fileExists('install.log')) {


                    log = readFile('install.log')


                }

                else if(fileExists('docker.log')) {


                    log = readFile('docker.log')


                }

                else if(fileExists('run.log')) {


                    log = readFile('run.log')


                }

                else if(fileExists('app.log')) {


                    log = readFile('app.log')


                }

                else {


                    log = currentBuild.rawBuild
                    .getLog(300)
                    .join("\n")


                }





                writeFile(

                    file: "failure.log",

                    text: log

                )





                sh '''
                /usr/bin/bash -c "


                PROMPT=\$(tr '\\n' ' ' < failure.log)



                curl -s --max-time 120 \
                http://host.docker.internal:11434/api/generate \
                -H 'Content-Type: application/json' \
                -d \"{

                \\\"model\\\":\\\"smollm2:latest\\\",

                \\\"prompt\\\":\\\"You are a Senior DevOps Engineer. Analyze this Jenkins failure. Failed Stage: ${FAILURE_STAGE}. Logs: \$PROMPT. Provide Build Status, Failed Stage, Root Cause, Exact Error, Why it Happened, Recommended Fix, Severity and Confidence Score.\\\",

                \\\"stream\\\":false

                }\" | python3 -c \"

                import sys,json

                data=json.load(sys.stdin)

                print(data.get('response',data))

                "



                "

                '''


                echo "Deployment Failed"


            }


        }


    }


}
