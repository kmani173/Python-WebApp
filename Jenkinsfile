pipeline {

    agent any


    environment {

        FAILURE_STAGE = "Not Identified"

    }


    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

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


                    env.FAILURE_STAGE="Build Docker Image"



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


                    env.FAILURE_STAGE="Run Container"



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


                    env.FAILURE_STAGE="Application Test"



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



        stage('AI Deployment Summary') {


            steps {


                sh '''
                /usr/bin/bash <<'EOF'


echo "===================================="

echo "AI DEPLOYMENT SUMMARY"

echo "===================================="



python3 <<'PY'


import json
import urllib.request



prompt="""

Deployment completed successfully.

Application:
Python-WebApp


Docker Image:
python-webapp:1.0


Container:
flask-demo


Generate deployment summary and recommendations.

"""



payload={

"model":"smollm2:latest",

"prompt":prompt,

"stream":False

}



req=urllib.request.Request(

"http://host.docker.internal:11434/api/generate",

data=json.dumps(payload).encode(),

headers={"Content-Type":"application/json"}

)



response=urllib.request.urlopen(req,timeout=120)

result=json.loads(response.read())


print(result.get("response"))


PY


EOF

                '''

            }

        }



    }





    post {


        failure {


            script {


                echo """

====================================
AI ROOT CAUSE ANALYSIS
====================================

"""


                def failedLog=""


                if(fileExists("install.log")) {

                    failedLog=readFile("install.log")

                }

                else if(fileExists("docker.log")) {

                    failedLog=readFile("docker.log")

                }

                else if(fileExists("run.log")) {

                    failedLog=readFile("run.log")

                }

                else if(fileExists("app.log")) {

                    failedLog=readFile("app.log")

                }

                else {

                    failedLog=currentBuild.rawBuild
                    .getLog(200)
                    .join("\n")

                }



                writeFile(

                    file:"failure.log",

                    text:failedLog

                )


                writeFile(

                    file:"failed-stage.txt",

                    text:env.FAILURE_STAGE

                )





                sh '''
                /usr/bin/bash <<'EOF'


python3 <<'PY'


import json
import urllib.request



with open("failure.log") as f:

    logs=f.read()



with open("failed-stage.txt") as f:

    stage=f.read()



prompt=f"""

You are a Senior DevOps Engineer.


Analyze Jenkins pipeline failure.



Return response exactly in this format:


====================================
AI ROOT CAUSE ANALYSIS
====================================


Build Status:
FAILED


Failed Stage:
{stage}


Exact Error:
<extract exact error from logs>


Root Cause:
<explain actual reason>


Why it Happened:
<technical explanation>


Recommended Fix:
<fix steps>


Severity:
LOW/MEDIUM/HIGH


Confidence Score:
percentage


====================================



Pipeline Logs:


{logs}


"""




payload={


"model":"smollm2:latest",

"prompt":prompt,

"stream":False


}



req=urllib.request.Request(

"http://host.docker.internal:11434/api/generate",

data=json.dumps(payload).encode(),

headers={"Content-Type":"application/json"}

)



try:


    response=urllib.request.urlopen(req,timeout=120)


    result=json.loads(response.read())


    print(result.get("response"))



except Exception as e:


    print("AI Analysis Failed")

    print(e)



PY


EOF

                '''



            }

        }



        success {


            echo """

====================================
DEPLOYMENT SUCCESSFUL
====================================

"""


        }


    }


}
