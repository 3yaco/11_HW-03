pipeline {
    agent any
    environment {
        IMAGE_NAME = 'ci-nginx-test'
        HOST_PORT = '9889'
        GIT_REPO = 'https://github.com/<USERNAME>/ci-nginx-demo.git'
    }
    stages {
        stage('Checkout') {
            steps {
                git url: "${GIT_REPO}"
            }
        }
        stage('Build Nginx Container') {
            steps {
                sh """
                docker build -t ${IMAGE_NAME} .
                docker run -d --name ${IMAGE_NAME}_container -p ${HOST_PORT}:80 ${IMAGE_NAME}
                """
            }
        }
        stage('Test HTTP Response') {
            steps {
                script {
                    def status = sh(script: "curl -s -o /dev/null -w '%{http_code}' http://localhost:${HOST_PORT}", returnStdout: true).trim()
                    if (status != '200') {
                        error "HTTP response code is ${status}, expected 200"
                    }
                }
            }
        }
        stage('Check MD5') {
            steps {
                script {
                    sh """
                    md5_local=\$(md5sum index.html | awk '{print \$1}')
                    md5_remote=\$(curl -s http://localhost:${HOST_PORT}/index.html | md5sum | awk '{print \$1}')
                    if [ "\$md5_local" != "\$md5_remote" ]; then
                        echo "MD5 mismatch!"
                        exit 1
                    fi
                    """
                }
            }
        }
    }
    post {
        always {
            sh "docker rm -f ${IMAGE_NAME}_container || true"
        }
        failure {
            sh 'curl -s -X POST https://api.telegram.org/bot<token>/sendMessage -d chat_id=<chat_id> -d text="CI failed!"'
        }
    }
}
