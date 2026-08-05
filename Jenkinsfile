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



prompt = """
You are a DevOps Engineer.

Generate ONLY the following deployment report.

=========================================================

AI DEPLOYMENT SUMMARY

=========================================================

Pipeline Status:
SUCCESS

Application:
Python-WebApp

Docker Image:
python-webapp:1.0

Container:
flask-demo

Health Check:
PASSED

Deployment Summary:
Describe the deployment in 3 short points.

Recommendation:
One sentence.

=========================================================

Return ONLY the report.
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



stage = "${FAILURE_STAGE}"



patterns = [
    r"ERROR:.*",
    r"error:.*",
    r"failed.*",
    r"Exception:.*",
    r"Traceback.*",
    r"docker:.*",
    r"curl:.*",
    r"permission denied.*",
    r"No such file.*",
    r"Cannot connect.*",
    r"returned a non-zero code.*",
    r"unable to.*"
]

errors = []

for p in patterns:
    errors.extend(re.findall(p, logs, re.IGNORECASE))

if errors:
    exact = " | ".join(dict.fromkeys(errors))[:1000]
else:
    exact = "No exact error found."

team = ""
prompt = f"""
You are a Senior DevOps Engineer.

Analyze the Jenkins pipeline logs.

Return ONLY the report in EXACTLY this format.

=======================================================

Pipeline Status:

FAILED

Root Cause:

Explain the actual technical root cause in 2 or 3 sentences.

Responsible Team:

Identify the responsible team based only on the error and logs.

Suggested Fix:

1. Give the first resolution step.

2. Give the second resolution step.

3. Give the third resolution step.

=======================================================

Exact Error:

{exact}

Complete Jenkins Logs:

{logs}

Rules:

- Analyze the Exact Error first.
- If Docker daemon/socket/build/run issues are found, select Docker Team.
- If pip/requirements/package installation issues are found, select Python Team.
- If curl or application health check fails, select Application Team.
- If Jenkins agent/plugin/workspace issues are found, select Jenkins Team.
- If SSH/VM/Linux permission issues are found, select Infrastructure Team.
- Base your answer ONLY on the logs.
- Do not guess.
- Return ONLY the report.
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

    response = urllib.request.urlopen(req, timeout=120)

    result = json.loads(response.read())

    ai_response = result.get("response", "")

    print("=" * 65)
    print("          AI ROOT CAUSE ANALYSIS REPORT")
    print("=" * 65)

    

    print(ai_response)

    print("=" * 65)

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
