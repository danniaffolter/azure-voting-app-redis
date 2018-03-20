#!/usr/bin/env groovy

pipeline {

    agent any
    options {
        timeout(time:5, unit: 'MINUTES')
    }
    environment {
        GITHASH = "_"
        SLACK_CHANNEL = "#clj12-k8s-cicd"
        WEB_IMAGE_NAME="${env.ACR_LOGINSERVER}/azure-vote-front:kube${env.BUILD_NUMBER}"
    }
    stages {
        
        stage('Preparation') {
            steps {
                echo('Get code from Github')
                // git 'https://github.com/danniaffolter/azure-voting-app-redis.git'
            }
        }
        
        stage('Compile Source Code') {
            steps {
                echo('compile source code')
            }
        }
        
        stage('Build and Push Image to ACR') {
            steps {
                echo('docker build, and push image to ACR')
                withCredentials([usernamePassword(credentialsId: 'acr-credentials', passwordVariable: 'ACR_PASSWORD', usernameVariable: 'ACR_ID')]) {
                    sh("docker build -t ${WEB_IMAGE_NAME} ./azure-vote")
                    sh("docker login ${env.ACR_LOGINSERVER} -u ${ACR_ID} -p ${ACR_PASSWORD}")
                    sh("docker push ${WEB_IMAGE_NAME}")
                }
            }
        }
        
        stage('Deploy to ACS') {
            steps {
                echo('update image on K8S')
                // deploy the app to k8s in default namespace
                // sh("kubectl create -f azure-vote-all-in-one-redis.yaml --namespace=default")
                sh("kubectl set image deployment/azure-vote-front azure-vote-front=${WEB_IMAGE_NAME} --kubeconfig /usr/local/bin/kube-config --namespace=default")
            }
        }  
    }
    post {
        success {
            script {
                SLACK_MESSAGE = "SUCCESS jenkins ${GITHASH}\nBuild ${env.JOB_NAME} ${currentBuild.displayName} completed in ${currentBuild.durationString.replace(' and counting','')} (<${currentBuild.absoluteUrl}|Open>)"
            }
            slackSend channel: "${SLACK_CHANNEL}", color: 'good', message: "${SLACK_MESSAGE}", teamDomain: 'starbucks', tokenCredentialId: 'slack-jenkins-token'
        }
        failure {
            script {
                SLACK_MESSAGE = "FAILURE jenkins ${GITHASH}\nBuild ${env.JOB_NAME} ${currentBuild.displayName} completed in ${currentBuild.durationString.replace(' and counting','')} (<${currentBuild.absoluteUrl}|Open>)"
            }
            slackSend channel: "${SLACK_CHANNEL}", color: 'danger', message: "${SLACK_MESSAGE}", teamDomain: 'starbucks', tokenCredentialId: 'slack-jenkins-token'


        }
        aborted {
            script {
                SLACK_MESSAGE = "ABORTED githash ${GITHASH}\nBuild ${env.JOB_NAME} ${currentBuild.displayName} completed in ${currentBuild.durationString.replace(' and counting','')} (<${currentBuild.absoluteUrl}|Open>)"
            }
                  slackSend channel: "${SLACK_CHANNEL}", color: 'warning', message: "${SLACK_MESSAGE}", teamDomain: 'starbucks', tokenCredentialId: 'slack-jenkins-token'
       }
    }
    
}
