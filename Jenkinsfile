#!/usr/bin/env groovy

pipeline {
    agent {
      docker {
        image 'node:6-alpine'
        args '-p 3000:3000'
      }
    }
    options {
      skipStagesAfterUnstable()
      disableConcurrentBuilds()
      timestamps()
      buildDiscarder(logRotator(numToKeepStr: '15', artifactNumToKeepStr: '15'))
    }

    parameters {
      string(name: 'BRANCH_NAME', defaultValue: 'master', description: 'Define branch name')
      string(name: 'ACCOUNT_ID', defaultValue: '', description: 'AccountId')
      choice(name: 'DEPLOY_ENV', choices: ['dev','staging','prod'], description: 'Define environment name')
    }

    environment {
        GIT_URL    = 'https://github.com/nguyenlelongtan/'
        GIT_REPO   = 'jenkinsdemo.git'
        GIT_CREDS  = 'tannll-account-github'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout( [$class: 'GitSCM',
                    branches: [[name: "*/${params.BRANCH_NAME}"]],
                    clearWorkspace: true,
                    clean: true,
                    userRemoteConfigs: [
                        [
                            credentialsId: "${GIT_CREDS}",
                            url: "${GIT_URL}/${GIT_REPO}"]
                        ]
                ])
            }
        }

        stage('STEP 0: Approve Deployment') {
            when {
                beforeAgent true
                expression { params.DEPLOY_ENV == 'prod' }
            }
            steps {
                timeout(time:1, unit:'DAYS') {
                    input message:'Approve deployment?'
                }
                echo 'Continue'
            }
        }

        stage("STEP 1: Get ACCOUNT_ID") {
           when {
             expression {
               currentBuild.result == null || currentBuild.result == 'SUCCESS'
             }
           }
           steps {
               script {
                   if ("${params.DEPLOY_ENV}" == 'dev') {
                       ACCOUNT_ID = params.ACCOUNT_ID
                   } else if ("${params.DEPLOY_ENV}" == 'staging') {
                       ACCOUNT_ID = params.ACCOUNT_ID
                   } else if ("${params.DEPLOY_ENV}" == 'prod') {
                       ACCOUNT_ID = params.ACCOUNT_ID
                   }
               }
           }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}
