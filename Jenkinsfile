pipeline{
    agent{
        label "master"
    }
    environment {
        GITHUB_URL = "https://github.com/GMKBabu/jenkins.git"
        GITHUB_CREDENTIALS = "cdb56ac9-d618-4df0-a85f-c41eb9647ef3"
        CUSTOM_TAG = "${COMMIT_ID}"
        IMAGE_NAME = "${IMAGE_REPO_NAME}:${CUSTOM_TAG}"
        CUSTOM_BUILD_NUMBER = "DEV-PRD-${BUILD_NUMBER}"
        GIT_COMMIT_HASH = sh (script: "git log -n 1 --pretty=format:'%H'", returnStdout: true)
        GIT_COMMIT_MESSAGE = sh (script: "git log -n 1 --pretty=format:'%s'", returnStdout: true)
        GIT_COMMIT_AUTHOR = sh (script: "git log -n 1 --pretty=format:'%an'", returnStdout: true)
    }
    parameters {
        string (name: 'GITHUB_BRANCH_NAME', defaultValue: 'master', description: 'Git branch to build')
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
        stage("Source Code Checkout"){
            steps{
                echo "========executing Source Code Checkout========"
                // using for checkout the code from bitbucket
                checkout([$class: 'GitSCM', branches: [[name: "*/${GITHUB_BRANCH_NAME}"]],doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], \
                   userRemoteConfigs: [[credentialsId: "${GITHUB_CREDENTIALS}", url: "${GITHUB_URL}"]]])
                script {
                    currentBuild.displayName = "${CUSTOM_BUILD_NUMBER}"
                    }
            }
        }

        stage("Build Docker Image"){
            steps{
                echo "====executing Build Docker Image===="

            }
        }

        stage("Approval") {
            steps{
             script {
                echo "====Waiting for Approval===="
               // GIT_COMMIT_HASH = sh (script: "git log -n 1 --pretty=format:'%H'", returnStdout: true)
               // GIT_COMMIT_MESSAGE = sh (script: "git log -n 1 --pretty=format:'%s'", returnStdout: true)
                echo "git commit id ${GIT_COMMIT_HASH}"
                echo "git commit message ${GIT_COMMIT_MESSAGE}"
                echo "git commit author ${GIT_COMMIT_AUTHOR}"
                emailext mimeType: 'text/html',
                         subject: "[Jenkins-Deploy-Approval]${currentBuild.fullDisplayName}",
                         to: "babu.g3090@gmail.com",
                         attachLog: true,
                         body: """<style>
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
                                         <th align="left"><strong>BUILD SUCCESS</strong></th>
                                      </tr>
                                    </table>
                                    <p><strong>Build URL:</strong><a href="${BUILD_URL}input">click to approve</a></p>
                                    <p><strong>Project:</strong> ${PROJECT_NAME}</p>
                                    <p><strong>Date of Build:</strong> <span id="dtText"></span></p>
                                    <p><strong>Build Duration:</strong> ${BUILD_DURATION}</p>
                                    <p style="border: 0px solid black;background-color:blue;color:white;" bgcolor="blue"><strong>CHANGES:</strong></p>
                                    <p> &#9658; <span id="Babu"></span></P>
                                    <script>
                                       var y=10;
                                       var x=10;
                                       if ( x == y ) {
                                             document.getElementById('Babu').innerHTML="No changes";
                                                } else {
                                                     document.getElementById('Babu').innerHTML="${GIT_COMMIT_MESSAGE}";
                                        }
                                    </script>
                                    <script>
                                         var today = new Date();
                                         document.getElementById('dtText').innerHTML=""+today;</script>
                                         <p style="border: 0px solid black;background-color:blue;color:white;" bgcolor="blue"><strong>BUILD ARTIFACTS:</strong></p>
                                    <table>
                                          <tr style="background-color:white;color:black;">
                                              <th>&#9658;</th>
                                              <th style="text-decoration: underline;color:blue;"><strong><a href="279944003491.dkr.ecr.eu-west-1.amazonaws.com/kubernates:${BUILD_NUMBER}"</a>279944003491.dkr.ecr.eu-west-1.amazonaws.com/kubernates:${BUILD_NUMBER}</strong></th>
                                          </tr>
                                    </table>
                                    <p style="border: 0px solid black;background-color:blue;color:white;" bgcolor="blue"><strong>BUILD INFORMATION:</strong></p>
                                    <table id="customers" style="width:100%;border: 1px solid black;border-collapse: collapse;">
                                        <tr style="border: 2px solid black;background-color:blue;color:white;">
                                            <th id="tableheader" style="width:30%;border: 1px solid black;border-collapse: collapse;" >BUILD</th>
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
                                            <td><strong>Last Successfull Commit:</strong></td>
                                            <td>sfsnfsfsncnjsjhfhlqwdllkdjasdhdw</td>
                                        </tr>
                                        <tr>
                                            <td><strong>Build Number:</strong></td>
                                            <td>${currentBuild.fullDisplayName}</td>
                                        </tr>
                                        <tr>
                                            <td><strong>Triggered by:</strong></td>
                                            <td>babu</td>
                                        </tr>
                                    </table>
                                </body>"""
                def userInput = input id: 'userInput',
                          message: 'Let\'s promote?', 
                          submitterParameter: 'submitter',
                          submitter: 'GMKBabu',
                          parameters: [
                              [$class: 'TextParameterDefinition', defaultValue: 'prod', description: 'Environment', name: 'env'],
                              [$class: 'TextParameterDefinition', defaultValue: 'k8s', description: 'Target', name: 'target']]
                echo ("Env: "+userInput['env'])
                echo ("Target: "+userInput['target'])
                echo ("submitted by: "+userInput['submitter'])
             }
            }
        }
        stage("Deploy") {
            steps{
                // uses https://plugins.jenkins.io/lockable-resources
                lock(resource: 'deployApplication') {
                    echo 'Deploying...'
                }
            }
        }
    }
    post{
        always{
            echo "========always========"
        }
        success{
            echo "========pipeline executed successfully ========"
        }
        failure{
            echo "========pipeline execution failed========"
        }
    }
}