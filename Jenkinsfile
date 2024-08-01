pipeline {
    agent any
    parameters {
        booleanParam(name: 'autoApprove', defaultValue: false, description: 'Automatically run apply after generating plan?')
    }
    environment {
        DOCKERHUB_CREDENTIALS = credentials('etienne-dockerhub')
    }
    stages {
        stage('Build docker image') {
            steps {
                sh 'docker build -t 9722411/flask:$BUILD_NUMBER .'
            }
        }
        stage('Login to Docker Hub') {
            steps {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }
        stage('Push Image') {
            steps {
                sh 'docker push 9722411/flask:$BUILD_NUMBER'
            }
        }
    post {
        always {
            sh 'docker logout'
        }
    }
