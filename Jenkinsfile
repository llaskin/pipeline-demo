#!/usr/bin/env groovy

node('master'){
  // def props = readProperties  file: 'project.properties'
  // def version = props.version
  def branch = "${env.BRANCH_NAME}"

  environment {
    SAUCE_CONNECT_TUNNEL = 'myTunnel'
  }

  echo branch
  // echo version

  stage('Checkout') {
    checkout scm
  }

  stage('Build') {
    sh 'npm install'
  }

  stage('Unit Tests') {
    sauce('derek_sauce_key') {
      withCredentials([usernamePassword(credentialsId: 'derek_sauce_key', passwordVariable: 'SAUCE_ACCESS_KEY', usernameVariable: 'SAUCE_USERNAME')]) {
        sauceconnect(options: '-u $SAUCE_USERNAME -k $SAUCE_ACCESS_KEY -i $SAUCE_CONNECT_TUNNEL', sauceConnectPath: '') {
          sh 'npm run test-single-run'
        }
      }
    }
  }

  if(env.BRANCH_NAME == 'develop') {
    stage('Deploy to Dev') {
      withCredentials([usernamePassword(credentialsId: 'derek_pcf', passwordVariable: 'CF_PW', usernameVariable: 'CF_USER')]) {
        sh 'cf login -a api.run.pivotal.io -u $CF_USER -p $CF_PW -o pipeline_demos -s development'
        sh 'cf push -f config/dev/manifest.yml'
      }
    }

    stage('E2E Tests') {
      withCredentials([usernamePassword(credentialsId: 'derek_sauce_key', passwordVariable: 'SAUCE_ACCESS_KEY', usernameVariable: 'SAUCE_USERNAME')]) {
        sh 'npm run protractor'
      }
    }
  }

  if(env.BRANCH_NAME.startsWith('release/')) {
    stage('Deploy to Prod') {
      withCredentials([usernamePassword(credentialsId: 'derek_pcf', passwordVariable: 'CF_PW', usernameVariable: 'CF_USER')]) {
        sh 'cf login -a api.run.pivotal.io -u $CF_USER -p $CF_PW -o pipeline_demos -s production'
        sh 'cf push -f config/prod/manifest.yml'
      }
    }
  }
}
