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
                emailext mimeType: 'text/html',
                         subject: "[Jenkins]${currentBuild.fullDisplayName}",
                         to: "babu.g3090@gmail.com",
                         body: '''<style>
                            body, table, td, th, p {
                                font-family:verdana,helvetica,sans serif;
                                font-size:11px;
                                color:black;
                                }
                            td.bg1 { color:white; background-color:#595959; font-size:120% }
                            td.console { font-family:courier new, lucida console; }
                            </style>
                            <body>
                                <table border=2 cellspacing=2 cellpadding=2 width="40%">
                                    <tr>
                                        <td align="left" width="40%">
                                           <b style="font-size: 170%;">GMKBabu</b>
                                        </td>
                                        <td valign="center" width="60%">
                                           <b style="font-size: 150%;">CICD Pipeline Information</b>
                                        </td>
                                    </tr>
                                    <tr>
                                       <td>URL:</td>
                                       <td><a href='${BUILD_URL}'>${JOB_NAME}</a></td>
                                    </tr>
                                    <tr>
                                        <td>BuildNumber:</td>
                                        <td>${CUSTOM_BUILD_NUMBER}</td>
                                    </tr>
                                    <tr>
                                        <td>JobName:</td>
                                        <td>${JOB_NAME}</td>
                                    </tr>
                                </table>
                              <br />
                            </body>'''
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
                 echo 'Deploying...'
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