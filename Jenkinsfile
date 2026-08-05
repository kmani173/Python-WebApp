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

team = ""
prompt = f"""
You are a Senior DevOps Engineer.

Analyze the Jenkins pipeline logs carefully.

Read the Exact Error first.

Then read the Complete Jenkins Logs.

Return ONLY the report below.

=======================================================

Pipeline Status:

FAILED

Failed Stage:

{stage}

Root Cause:

Explain ONLY the actual technical reason for the failure using the logs.

Responsible Team:

Choose ONLY ONE:

Docker Team
Python Team
Jenkins Team
Infrastructure Team
Application Team
Network Team
DevOps Team

Suggested Fix:

1.

2.

3.

=======================================================

Exact Error:

{exact}

Complete Jenkins Logs:

{logs}

Rules:

1. Read the Exact Error first.

2. Then verify using the Complete Jenkins Logs.

3. If the failure is related to docker build, docker run, docker daemon or docker.sock, return Docker Team.

4. If the failure is related to pip, requirements.txt, package installation or PyPI, return Python Team.

5. If the failure is related to curl or application health check, return Application Team.

6. If the failure is related to Jenkins agent, plugins or workspace, return Jenkins Team.

7. If the failure is related to Linux permissions, SSH or VM access, return Infrastructure Team.

8. Do not invent errors.

9. Do not return "None".

10. Do not say "No specific root cause".

11. Use the Exact Error as the primary source.

12. Return ONLY the report.
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
