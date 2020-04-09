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
                echo "====Waiting for Approval===="
                 def deploymentDelay = input id: 'Deploy', message: 'Deploy to production?', submitter: 'rkivisto,admin', parameters: [choice(choices: ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9', '10', '11', '12', '13', '14', '15', '16', '17', '18', '19', '20', '21', '22', '23', '24'], description: 'Hours to delay deployment?', name: 'deploymentDelay')]
                 sleep time: deploymentDelay.toInteger(), unit: 'HOURS'
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