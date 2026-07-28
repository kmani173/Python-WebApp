pipeline {

    agent any


    environment {

        FAILURE_STAGE = "Unknown"

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

                    env.FAILURE_STAGE = "Build Docker Image"


                    sh '''
                    /usr/bin/bash -c "


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


                    sleep 10


                    curl -f \
                    http://host.docker.internal:5000 \
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


python3 <<'PY'


import json
import urllib.request



prompt="""

Application Deployment Successful.

Application:
Python-WebApp


Docker Image:
python-webapp:1.0


Container:
flask-demo


Provide deployment summary.


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




                sh '''
                /usr/bin/bash <<'EOF'


python3 <<'PY'


import json
import urllib.request
import re



with open("failure.log") as f:

    logs=f.read()



stage="${FAILURE_STAGE}"



errors=re.findall(

r"(ERROR:.*|Exception:.*|failed:.*|No matching.*)",

logs

)



if errors:

    exact=" ".join(errors)

else:

    exact="Unable to extract error"



prompt=f"""


You are a Senior DevOps Engineer.


Analyze Jenkins pipeline failure.



Return exactly:



====================================
AI ROOT CAUSE ANALYSIS
====================================


Build Status:
FAILED


Failed Stage:
{stage}


Exact Error:
{exact}


Root Cause:
Explain the technical root cause.


Why it Happened:
Explain why this occurred.


Recommended Fix:
Provide exact remediation steps.


Severity:
LOW/MEDIUM/HIGH


Confidence Score:
percentage



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


    print("AI RCA failed:")

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
