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
                /usr/bin/bash <<EOF

                export FAILURE_STAGE="$FAILURE_STAGE"
                python3 <<'PY'


import json
import urllib.request
import re



with open("failure.log") as f:

    logs=f.read()



import os

stage = os.getenv("FAILURE_STAGE", "Unknown")



patterns = [
    r"permission denied.*",
    r"docker:.*",
    r"ERROR:.*",
    r"No matching distribution.*",
    r"Could not find a version.*",
    r"Cannot connect.*",
    r"connection refused.*",
    r"curl:.*",
    r"Traceback.*",
    r"Exception.*",
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

lower_logs = (exact + "\n" + logs).lower()

if (
    "docker.sock" in lower_logs or
    "docker daemon" in lower_logs or
    "permission denied while trying to connect to the docker daemon" in lower_logs or
    "docker build" in lower_logs or
    "docker run" in lower_logs
):
    team = "Docker Team"

elif (
    "requirements.txt" in lower_logs or
    "pip install" in lower_logs or
    "no matching distribution found" in lower_logs or
    "could not find a version that satisfies the requirement" in lower_logs
):
    team = "Python Team"

elif (
    "curl:" in lower_logs or
    "connection refused" in lower_logs or
    "5000" in lower_logs
):
    team = "Application Team"

elif (
    "plugin" in lower_logs or
    "workspace" in lower_logs or
    "hudson" in lower_logs or
    "jenkins" in lower_logs
):
    team = "Jenkins Team"

elif (
    "permission denied" in lower_logs or
    "ssh" in lower_logs
):
    team = "Infrastructure Team"

else:
    team = "DevOps Team"

logs = logs[:4000]
prompt = f"""
You are a Senior DevOps Engineer.

The Responsible Team has ALREADY been determined by the pipeline.

DO NOT classify the team.
DO NOT change the team.
Copy it exactly.

Analyze ONLY:

1. Failed Stage
2. Root Cause
3. Suggested Fix

The Responsible Team MUST remain exactly as supplied below.

Return ONLY this format.

=======================================================

Pipeline Status:

FAILED

Failed Stage:

{stage}

Root Cause:

Explain ONLY the actual technical reason for the failure using the logs.

Responsible Team:

{team}

IMPORTANT:
Do NOT change the Responsible Team above.
Print it exactly as provided.

Suggested Fix:

1.

2.

3.

=======================================================

Exact Error:

{exact}

Complete Jenkins Logs:

{logs[:2500]}

Rules:

- Use the Responsible Team exactly as provided.
- Do not change the Responsible Team.
- Do not output None.
- Do not invent another team.
- Use the Exact Error as the primary root cause.
- Do not recommend unrelated fixes.
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
