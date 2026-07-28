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

                catchError(buildResult: 'FAILURE', stageResult: 'FAILURE') {

                    script {

                        env.FAILURE_STAGE="Verify Python"

                        sh '''
                        /usr/bin/bash -c "

                        python3 --version
                        pip3 --version

                        "
                        '''

                    }

                }

            }

        }



        stage('Install Dependencies') {

            steps {

                catchError(buildResult:'FAILURE', stageResult:'FAILURE') {

                    script {


                        env.FAILURE_STAGE="Install Dependencies"


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

        }




        stage('Build Docker Image') {


            steps {


                catchError(buildResult:'FAILURE', stageResult:'FAILURE') {


                    script {


                        env.FAILURE_STAGE="Build Docker Image"


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


        }




        stage('Run Container') {


            steps {


                catchError(buildResult:'FAILURE', stageResult:'FAILURE') {


                    script {


                        env.FAILURE_STAGE="Run Container"



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


        }




        stage('Application Test') {


            steps {


                catchError(buildResult:'FAILURE', stageResult:'FAILURE') {


                    script {


                        env.FAILURE_STAGE="Application Test"


                        sh '''
                        /usr/bin/bash -c "

                        sleep 10


                        curl -f http://host.docker.internal:5000 \
                        2>&1 | tee app.log


                        "
                        '''


                    }


                }


            }


        }


    }



    post {


        always {


            script {


                if(currentBuild.currentResult == 'FAILURE') {



                    echo """

====================================
AI ROOT CAUSE ANALYSIS
====================================

"""



                    def logfile=""


                    if(fileExists("install.log")) {

                        logfile=readFile("install.log")

                    }

                    else if(fileExists("docker.log")) {

                        logfile=readFile("docker.log")

                    }

                    else if(fileExists("run.log")) {

                        logfile=readFile("run.log")

                    }

                    else if(fileExists("app.log")) {

                        logfile=readFile("app.log")

                    }



                    writeFile(

                        file:"failure.log",

                        text:logfile

                    )



                    sh '''
                    /usr/bin/bash <<'EOF'


python3 <<'PY'


import json
import urllib.request
import re



with open("failure.log") as f:

    logs=f.read()



errors=re.findall(

r"(ERROR:.*|Exception:.*|failed:.*|No matching.*)",

logs

)



if errors:

    error_text=" ".join(errors)

else:

    error_text="Unable to extract exact error"



prompt=f"""


You are a Senior DevOps Engineer.



Analyze Jenkins failure.



Return exactly:



====================================
AI ROOT CAUSE ANALYSIS
====================================


Build Status:
FAILED


Failed Stage:
Install Dependencies


Exact Error:
{error_text}


Root Cause:
Explain technical root cause.


Why it Happened:
Explain reason.


Recommended Fix:
Provide solution steps.


Severity:
HIGH/MEDIUM/LOW


Confidence Score:
percentage



Logs:

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



response=urllib.request.urlopen(req,timeout=120)


result=json.loads(response.read())


print(result.get("response"))



PY


EOF

                    '''



                }

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
