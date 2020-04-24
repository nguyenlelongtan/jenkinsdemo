#!/usr/bin/env groovy

pipeline {
    agent { label 'agent-alpine-node-py-stable-xl' }

    options {
      skipStagesAfterUnstable()
      disableConcurrentBuilds()
      timestamps()
      buildDiscarder(logRotator(numToKeepStr: '15', artifactNumToKeepStr: '15'))
    }

    parameters {
      string(name: 'BRANCH_NAME', defaultValue: 'master', description: 'Define branch name')
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
    }
    post {
        always {
            cleanWs()
        }
    }
}
