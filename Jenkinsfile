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
      string(name: 'ACCOUNT_ID', defaultValue: '', description: 'ACCOUNT_ID')
      string(name: 'DEPLOYER_ROLE', defaultValue: '', description: 'DEPLOYER_ROLE')
      string(name: 'S3_BUCKET_URL_CONFIG', defaultValue: '', description: 'S3_BUCKET_URL_CONFIG')
      string(name: 'S3_BUCKET_URL_STATIC_WEB', defaultValue: '', description: 'S3_BUCKET_URL_STATIC_WEB')
      string(name: 'AWS_REGION', defaultValue: '', description: 'AWS_REGION')
      string(name: 'AWS_ACCESS_KEY_ID', defaultValue: '', description: 'AWS_ACCESS_KEY_ID')
      string(name: 'AWS_SECRET_KEY', defaultValue: '', description: 'AWS_SECRET_KEY')

      choice(name: 'DEPLOY_ENV', choices: ['dev','staging','prod'], description: 'Define environment name')
    }

    environment {
        GIT_URL    = 'https://github.com/nguyenlelongtan'
        GIT_REPO   = 'jenkinsdemo.git'
        GIT_CREDS  = 'tannll-account-github'
        AWS_ACCESS_KEY_ID = "${params.AWS_ACCESS_KEY_ID}"
        AWS_SECRET_KEY = "${params.AWS_SECRET_KEY}"
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

        stage("STEP 2: Update/Deploy MDE oD WebUI") {
            when {
              expression {
                currentBuild.result == null || currentBuild.result == 'SUCCESS'
              }
            }
            steps {
                withAWS(region: "${params.AWS_REGION}", role: "arn:aws:iam::${ACCOUNT_ID}:role/${params.DEPLOYER_ROLE}") {
                  sh """
                  rm -rf webui
                  mkdir webui
                  npm install
                  cp -a build/. webui
                  """

                  zip dir: "webui", zipFile: "webui.0.${env.BUILD_NUMBER}.zip", archive: true

                  sh """
                  aws s3 sync --sse AES256 --exclude "*" --include "*.zip" ${env.WORKSPACE}/ "${params.S3_BUCKET_URL_CONFIG}/"
                  aws s3 sync --sse AES256 build "${params.S3_BUCKET_URL_STATIC_WEB}"
                  """
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
