pipeline {
    agent any

    environment {
        IMAGE_NAME = "ci-nginx-test"
        CONTAINER_NAME = "ci-nginx-test_container"
        HOST_PORT = "9889"
        CONTAINER_PORT = "80"
        INDEX_FILE = "index.html"
        MD5_LOCAL = sh(script: "md5sum ${INDEX_FILE} | awk '{print \$1}'", returnStdout: true).trim()
        TELEGRAM_TOKEN = credentials('8291555588:AAERSsfJ7eCyzE4JRKiHAt1F-zrigDJZvZw') // добавь Credential с токеном
        TELEGRAM_CHAT_ID = credentials('80683779') // добавь Credential с chat_id
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Nginx Container') {
            steps {
                script {
                    docker.build("${IMAGE_NAME}", ".")
                }
            }
        }

        stage('Run Container') {
            steps {
                script {
                    sh "docker run -d --name ${CONTAINER_NAME} -p ${HOST_PORT}:${CONTAINER_PORT} ${IMAGE_NAME}"
                }
            }
        }

        stage('Test HTTP Response') {
            steps {
                script {
                    def response = sh(script: "curl -s -o /dev/null -w '%{http_code}' http://localhost:${HOST_PORT}", returnStdout: true).trim()
                    if (response != "200") {
                        error "HTTP status is not 200, it is ${response}"
                    }
                }
            }
        }

        stage('Check MD5') {
            steps {
                script {
                    def md5_remote = sh(script: "curl -s http://localhost:${HOST_PORT}/${INDEX_FILE} | md5sum | awk '{print \$1}'", returnStdout: true).trim()
                    if (md5_remote != MD5_LOCAL) {
                        error "MD5 mismatch: local ${MD5_LOCAL}, remote ${md5_remote}"
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                sh "docker rm -f ${CONTAINER_NAME} || true"
            }
        }

        failure {
            script {
                sh """
                curl -s -X POST https://api.telegram.org/bot${TELEGRAM_TOKEN}/sendMessage \
                -d chat_id=${TELEGRAM_CHAT_ID} \
                -d text='CI failed!'
                """
            }
        }
    }
}
