pipeline {
    agent any
    parameters {
        booleanParam(name: 'autoApprove', defaultValue: false, description: 'Automatically run apply after generating plan?')
    }
    environment {
        DOCKERHUB_CREDENTIALS = credentials('etienne-dockerhub')
        TRIVY_VERSION = '0.42.0'
        AWS_ACCESS_KEY_ID = credentials('Access key ID')
        AWS_SECRET_ACCESS_KEY = credentials('Secret access key')
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
/*        stage('Trivy Scan') {
            steps {
                script {
                    def trivyInstalled = sh(script: 'which trivy', returnStatus: true) == 0
                    if (!trivyInstalled) {
                        sh """
                            wget https://github.com/aquasecurity/trivy/releases/latest/download/trivy_${TRIVY_VERSION}_Linux-64bit.deb
                            sudo dpkg -i trivy_${TRIVY_VERSION}_Linux-64bit.deb
                        """
                    }
                }
                sh 'trivy image --severity HIGH,CRITICAL 9722411/flask:$BUILD_NUMBER'
            }
        }*/
        stage('Checkout Terraform') {
            steps {
                script {
                    dir("terraform") {
                        git 'https://github.com/evauclin/docker-jenkins.git'
                    }
                }
            }
        }
        stage('Terraform Plan') {
            steps {
                script {
                    dir("terraform") {
                        sh 'terraform init'
                        sh 'terraform plan -out tfplan'
                        sh 'terraform show -no-color tfplan > tfplan.txt'
                    }
                }
            }
        }
        stage('Approval') {
            when {
                not {
                    equals expected: true, actual: params.autoApprove
                }
            }
            steps {
                script {
                    dir("terraform") {
                        def plan = readFile 'tfplan.txt'
                        input message: 'Do you want to apply the plan?',
                        parameters: [text(name: 'Plan', description: 'Please review the plan', defaultValue: plan)]
                    }
                }
            }
        }
        stage('Terraform Apply') {
            steps {
                script {
                    dir("terraform") {
                        sh 'terraform apply -input=false tfplan'
                    }
                }
            }
        }
    }
    post {
        always {
            sh 'docker logout'
        }
    }
}
