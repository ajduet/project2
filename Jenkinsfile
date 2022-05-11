pipeline {
  agent any
  stages {
    stage('Quality Gate') {
        steps{}
    }
    stage('Unit Testing') {
        when {
            anyOf {branch 'ft_*'; branch 'bg_*'}
        }
        steps{
            withMaven {
                sh 'mvn test'
            }
        }
    }
    stage('Build') {
        steps{}
    }
    stage('Docker Image') {
        steps{}
    }
    stage('Docker Deliver') {
        steps{}
    }
    stage('Wait for approval') {
        steps{}
    }
    stage('Deploy'){
        steps{}
    }
  }
}