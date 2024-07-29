pipeline {
    agent any
    environment {
        DOCKERHUB_CREDENTIALS = credentials('etienne-dockerhub')
        TRIVY_VERSION = '0.42.0'
    }
    stages {
        stage('Build docker image') {
            steps {
                sh 'docker build -t 9722411/flask:$BUILD_NUMBER .'
            }
        }
        stage('login to dockerhub') {
            steps {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }
        stage('push image') {
            steps {
                sh 'docker push 9722411/flask:$BUILD_NUMBER'
            }
        }
        stage('Trivy scan') {
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
        }
    }
    post {
        always {
            sh 'docker logout'
        }
    }
}
