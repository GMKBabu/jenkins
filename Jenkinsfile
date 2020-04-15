pipeline{
    agent{
        label "master"
    }
    environment {
        GITHUB_URL = "https://github.com/GMKBabu/jenkins.git"
        GITHUB_CREDENTIALS = "cdb56ac9-d618-4df0-a85f-c41eb9647ef3"
        DOCKERHUB_CREDENTIALS = "DockerHub"
        DOCKERHUB_REPOSITORY_URL = "https://index.docker.io/v1/"
        CUSTOM_TAG = "${BUILD_NUMBER}"
        IMAGE_REPO_NAME = "gmkbabu/test-cicd"
        IMAGE_NAME = "${IMAGE_REPO_NAME}:${CUSTOM_TAG}"
        CUSTOM_BUILD_NUMBER = "DEV-PRD-${BUILD_NUMBER}"
        ID = "cicd"
        TEST_LOCAL_PORT = "80"
        GIT_COMMIT_HASH = sh (script: "git log -n 1 --pretty=format:'%H'", returnStdout: true)
        GIT_COMMIT_MESSAGE = sh (script: "git log -n 1 --pretty=format:'%s'", returnStdout: true)
        GIT_COMMIT_AUTHOR = sh (script: "git log -n 1 --pretty=format:'%an'", returnStdout: true)
    }
    parameters {
        string (name: 'GITHUB_BRANCH_NAME', defaultValue: 'master', description: 'Git branch to build')
        //booleanParam (name: 'DEPLOY_TO_PROD', defaultValue: false, description: 'If build and tests are good, proceed and deploy to production without manual approval')
    }
    triggers {
        //Run Polling of GitHub every minute everyday of the week
        pollSCM ('* * * * *')
        //cron ('0 0 * * 1-5')
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '5', artifactDaysToKeepStr: '3', artifactNumToKeepStr: '1'))
        timeout(time: 60, unit: 'MINUTES')
    }
    // Pipeline stages
    stages{
        stage("show_user_name") {
            steps {
            script {
                   wrap([$class: 'BuildUser']) {
                       GIT_BUILD_USER = sh ( script: 'echo "${BUILD_USER}"', returnStdout: true).trim()
                   }
               }
            }
        }
        stage("Source Code Checkout"){
            steps{
                script {
                    echo "========executing Source Code Checkout========"
                    // using for checkout the code from bitbucket
                    def scmVars = checkout([$class: 'GitSCM', branches: [[name: "*/${GITHUB_BRANCH_NAME}"]],doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], \
                         userRemoteConfigs: [[credentialsId: "${GITHUB_CREDENTIALS}", url: "${GITHUB_URL}"]]])
                    
                    scmRevisionNumber = scmVars.GIT_COMMIT
                    scmPreviousCommit = scmVars.GIT_PREVIOUS_COMMIT
                    scmPreviousSuccessfulCommit = scmVars.GIT_PREVIOUS_SUCCESSFUL_COMMIT

                    currentBuild.description = "this is for testing"
                    currentBuild.displayName = "${CUSTOM_BUILD_NUMBER}"

                }

                println("GitHub Revision Number = ${scmRevisionNumber}")
                println("GitHub Revion Message = ${GIT_COMMIT_MESSAGE}")
                println("GitHub Author email = ${GIT_COMMIT_AUTHOR}")
                println("previous commit = ${scmPreviousCommit}")
                println("last successful Commit  = ${scmPreviousSuccessfulCommit}")
            }
        }

        stage("Build Docker Image and Test"){
            parallel {
                //Docker Image Build
                stage("Docker Image Build") {
                    steps {
                        echo "====executing Build Docker Image===="
                        sh "docker build -t $IMAGE_NAME  ${WORKSPACE}/."
                    }
                }

                //Run Docker Container through docker image
                stage("Docker Container") {
                    steps {
                        echo "====executing Build Docker Image TO Container===="
                        // Kill container in case there is a leftover
                        sh "[ -z \"\$(docker ps -a | grep ${ID} 2>/dev/null)\" ] || docker rm -f ${ID}"

                        echo "Starting ${IMAGE_REPO_NAME} container"
                        sh "docker run --detach --name ${ID} --rm --publish ${TEST_LOCAL_PORT}:80 ${IMAGE_NAME}"
                    }
                }

                //Docker Container Local Test
                stage("Local Test Docker Container") {
                    steps {
                        sh '''
                            host_ip=$(hostname -i)
				            curl -aG http://$host_ip:80
				        '''
                    }
                }
            }
        }

        stage("Push Image to Docker Hub") {
            parallel {
                stage("Stop Docker Container") {
                    steps {
                        echo "Stop the Docker Container"
                        sh 'docker stop "${ID}"'
                    }
                }
                stage("Push Docker Image to Docker Hub"){
                    steps {
                        withDockerRegistry(credentialsId: "${DOCKERHUB_CREDENTIALS}", url: "${DOCKERHUB_REPOSITORY_URL}") {
                            echo "====++++Push Docker Image to Docker Hub++++===="
                            sh 'docker push "${IMAGE_NAME}"'
                        }
                    }
                }
            }
        }

        stage("Approval") {
            steps{
             script {
                echo "====Waiting for Approval===="
                emailext mimeType: 'text/html',
                         subject: "[Jenkins-Deploy-Approval]${currentBuild.fullDisplayName}",
                         to: "babu.g3090@gmail.com",
                         attachLog: true,
                         body: """<!DOCTYPE html>
                               <html>
                               <head> 
                                  <style>
                                     #customers td, #customers th {
                                         border: 1px solid black;
                                         padding: 6px;
                                         border-collapse: collapse;
                                         }
                                      #tableheader th {
                                          font-weight: bold;
                                          }
                                      .footer {
                                          position: fixed;
                                          left: 30;
                                          right: 60;
                                          bottom: 20;
                                          width: 20%;
                                          color: black;
                                          text-align: left;
                                          }
                                    body{        
                                        padding-bottom: 400px;
                                        }
                                    </style>
                                    </head>
                                    <body>
                                    <table>
                                      <tr style="background-color:white;color:black;">
                                         <th width="10"><img src="http://i.imgur.com/uXlqCxW.gif" alt="Smiley face" height="30" width="30"></th>
                                         <th align="left"><strong>BUILD APPROVAL</strong></th>
                                      </tr>
                                    </table>
                                    <p><strong>Build URL:</strong><a href="${BUILD_URL}input">click to approve</a></p>
                                    <p><strong>Project:</strong> ${currentBuild.fullDisplayName}</p>
                                    <p><strong>Project:</strong> ${JOB_NAME}</p>
                                    <p><strong>Date of Build:</strong> <span id="dtText"></span> ${BUILD_TIMESTAMP}</p>
                                    <p><strong>Build Duration:</strong> ${currentBuild.durationString}</p>
                                    <p style="border: 0px solid black;background-color:blue;color:white;" bgcolor="blue"><strong>CHANGES:</strong></p>
                                    <p> &#9658; ${GIT_COMMIT_MESSAGE}</P>
                                    <script>
                                         var today = new Date();
                                         document.getElementById('dtText').innerHTML=""+today;</script>
                                         <p style="border: 0px solid black;background-color:blue;color:white;" bgcolor="blue"><strong>BUILD ARTIFACTS:</strong></p>
                                    <table>
                                          <tr style="background-color:white;color:black;">
                                              <th>&#9658;</th>
                                              <th style="text-decoration: underline;color:blue;"><strong><a href="https://hub.docker.com/repository/docker/gmkbabu/test-cicd:${BUILD_NUMBER}">https://hub.docker.com/repository/docker/gmkbabu/test-cicd:${BUILD_NUMBER}</a></strong></th>
                                          </tr>
                                    </table>
                                    <p style="border: 0px solid black;background-color:blue;color:white;" bgcolor="blue"><strong>BUILD INFORMATION:</strong></p>
                                    <table id="customers" style="width:100%;border: 2px solid black;border-collapse: separate;">
                                        <tr style="border: 2px solid black;background-color:blue;color:white;">
                                            <th id="tableheader" style="width:30%;border: 2px solid black;border-collapse: collapse;" >BUILD</th>
                                            <th>DETAILS</th>
                                        </tr>
                                        <tr>
                                            <td><strong>Commit ID:</strong></td>
                                            <td>${GIT_COMMIT_HASH}</td>
                                        </tr>
                                        <tr>
                                            <td><strong>Commit Author:</strong></td
                                            <td>${GIT_COMMIT_AUTHOR}</td>
                                        </tr>
                                        <tr>
                                            <td><strong>Previous Successfull Commit:</strong></td>
                                            <td>${scmPreviousCommit}</td>
                                        </tr>
                                        <tr>
                                            <td><strong>Last Successfull Commit:</strong></td>
                                            <td>${scmPreviousSuccessfulCommit}</td>
                                        </tr>
                                        <tr>
                                            <td><strong>Build Number:</strong></td>
                                            <td>${currentBuild.fullDisplayName}</td>
                                        </tr>
                                        <tr>
                                            <td><strong>Triggered by:</strong></td>
                                            <td>${GIT_BUILD_USER}</td>
                                        </tr>
                                    </table>
                                </body>
                                <div class="footer">
                                <footer>
                                  <p><strong>thanks</strong></p>
                                  <p>DevOps Team</p>
                                </footer>
                                </div>
                                </html>"""
                def userInput = input id: 'userInput',
                          message: 'Let\'s promote?', 
                          submitterParameter: 'submitter',
                          submitter: 'GMKBabu',
                          parameters: [
                              [$class: 'BooleanParameterDefinition', defaultValue: false, description: 'Environment', name: 'DEPLOY_TO_PROD'],
                              [$class: 'TextParameterDefinition', defaultValue: 'k8s', description: 'Target', name: 'target']]
                
                echo ("DEPLOY_TO_PROD: "+userInput['DEPLOY_TO_PROD'])
                echo ("Target: "+userInput['target'])
                echo ("submitted by: "+userInput['submitter'])

                env.DEPLOY_TO_PROD = userInput.DEPLOY_TO_PROD
                env.BUILD_APPROVED = userInput.submitter
                echo "Selected Environment: ${DEPLOY_TO_PROD}"
             }
            }
        }
        stage("Deploy") {
            when {
                allOf {
                    environment name: 'GITHUB_BRANCH_NAME', value: 'master'
                    environment name: 'DEPLOY_TO_PROD', value: 'true'
                }
            }
            steps{
                // uses https://plugins.jenkins.io/lockable-resources
                lock(resource: 'deployApplication') {
                    echo 'Deploying...'

                }
            }
        }
    }
    post{
        success{
            echo "========pipeline executed successfully ========"
            script {

                currentBuild.result = "SUCCESS"
            }
            NotifyEmail()
        }
        failure{
            echo "========pipeline execution failed========"
            script {
                currentBuild.result = "FAILURE"
            }
            NotifyEmail()
        }
    }
}
def NotifyEmail() {
        emailext mimeType: 'text/html',
                   to: "babu.m@connectio.co.in",
                   subject: "Status: ${currentBuild.result}",
                   attachLog: true,
                   body: """<!DOCTYPE html>
                               <html>
                               <head>      
                                 <style>
                                     #customers td, #customers th {
                                         border: 1px solid black;
                                         padding: 6px;
                                         border-collapse: collapse;
                                         }
                                      #tableheader th {
                                          font-weight: bold;
                                          }
                                      .footer {
                                          position: fixed;
                                          left: 30;
                                          right: 60;
                                          bottom: 20;
                                          width: 20%;
                                          color: black;
                                          text-align: left;
                                          }
                                    body{        
                                        padding-bottom: 400px;
                                        }
                                    </style>
                                    </head>
                                    <body>
                                    <table>
                                      <tr style="background-color:white;color:black;">
                                         <th width="10"><img src="http://i.imgur.com/uXlqCxW.gif" alt="Smiley face" height="30" width="30"></th>
                                         <th align="left"><strong>BUILD ${currentBuild.result}</strong></th>
                                      </tr>
                                    </table>
                                    <p><strong>Build URL: </strong> ${BUILD_URL}</p>
                                    <p><strong>Project:</strong> ${currentBuild.fullDisplayName}</p>
                                    <p><strong>Project:</strong> ${JOB_NAME}</p>
                                    <p><strong>Date of Build:</strong> <span id="dtText"></span> ${BUILD_TIMESTAMP}</p>
                                    <p><strong>Build Duration:</strong> ${currentBuild.durationString}</p>
                                    <p style="border: 0px solid black;background-color:blue;color:white;" bgcolor="blue"><strong>CHANGES:</strong></p>
                                    <p> &#9658; ${GIT_COMMIT_MESSAGE}</P>
                                    <script>
                                         var today = new Date();
                                         document.getElementById('dtText').innerHTML=""+today;</script>
                                         <p style="border: 0px solid black;background-color:blue;color:white;" bgcolor="blue"><strong>BUILD ARTIFACTS:</strong></p>
                                    <table>
                                          <tr style="background-color:white;color:black;">
                                              <th>&#9658;</th>
                                              <th style="text-decoration: underline;color:blue;"><strong><a href="https://hub.docker.com/repository/docker/gmkbabu/test-cicd:${BUILD_NUMBER}">https://hub.docker.com/repository/docker/gmkbabu/test-cicd:${BUILD_NUMBER}</a></strong></th>
                                          </tr>
                                    </table>
                                    <p style="border: 0px solid black;background-color:blue;color:white;" bgcolor="blue"><strong>BUILD INFORMATION:</strong></p>
                                    <table id="customers" style="width:100%;border: 2px solid black;border-collapse: separate;">
                                        <tr style="border: 2px solid black;background-color:blue;color:white;">
                                            <th id="tableheader" style="width:30%;border: 2px solid black;border-collapse: collapse;" >BUILD</th>
                                            <th>DETAILS</th>
                                        </tr>
                                        <tr>
                                            <td><strong>Commit ID:</strong></td>
                                            <td>${GIT_COMMIT_HASH}</td>
                                        </tr>
                                        <tr>
                                            <td><strong>Commit Author:</strong></td
                                            <td>${GIT_COMMIT_AUTHOR}</td>
                                        </tr>
                                        <tr>
                                            <td><strong>Previous Successfull Commit:</strong></td>
                                            <td>${scmPreviousCommit}</td>
                                        </tr>
                                        <tr>
                                            <td><strong>Last Successfull Commit:</strong></td>
                                            <td>${scmPreviousSuccessfulCommit}</td>
                                        </tr>
                                        <tr>
                                            <td><strong>Build Number:</strong></td>
                                            <td>${currentBuild.fullDisplayName}</td>
                                        </tr>
                                        <tr>
                                            <td><strong>Triggered by:</strong></td>
                                            <td>${GIT_BUILD_USER}</td>
                                        </tr>
                                        <tr>
                                            <td><strong>Approved by:</strong></td>
                                            <td>${BUILD_APPROVED}</td>
                                        </tr>
                                    </table>
                                </body> 
                                <div class="footer">
                                <footer>
                                  <p><strong>thanks</strong></p>
                                  <p>DevOps Team</p>
                                </footer>
                                </div>
                                </html>"""
}