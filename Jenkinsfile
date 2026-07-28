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


            if (fileExists('install.log')) {

                log = readFile('install.log')

            }
            else if (fileExists('docker.log')) {

                log = readFile('docker.log')

            }
            else if (fileExists('run.log')) {

                log = readFile('run.log')

            }
            else if (fileExists('app.log')) {

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
            /usr/bin/bash <<'EOF'


echo "Sending failure logs to Ollama..."



python3 <<'PY'

import json
import urllib.request


with open("failure.log","r") as f:
    logs=f.read()


prompt=f"""
You are a Senior DevOps Engineer.

Analyze this Jenkins pipeline failure.

Provide:

1. Build Status
2. Failed Stage
3. Root Cause
4. Exact Error
5. Why it Happened
6. Recommended Fix
7. Severity
8. Confidence Score


Jenkins Logs:

{logs}

"""


payload={

"model":"smollm2:latest",

"prompt":prompt,

"stream":False

}



data=json.dumps(payload).encode("utf-8")


req=urllib.request.Request(

"http://host.docker.internal:11434/api/generate",

data=data,

headers={
"Content-Type":"application/json"
}

)



try:

    response=urllib.request.urlopen(req,timeout=120)

    result=json.loads(response.read())


    print(result.get("response","No AI response received"))


except Exception as e:

    print("AI Analysis Failed:")

    print(e)


PY


EOF
            '''


            echo "===================================="
            echo "Deployment Failed"
            echo "===================================="


        }

    }

}
}
    
    
