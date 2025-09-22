pipeline {
    agent any

    environment {
        CONTAINER_NAME = "ci-nginx-test"
        TELEGRAM_TOKEN = credentials('telegram_token')
        TELEGRAM_CHAT_ID = credentials('telegram_chat_id')
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
                    sh "docker build -t ${CONTAINER_NAME} ."
                    sh "docker run -d --name ${CONTAINER_NAME} -p 8080:80 ${CONTAINER_NAME}"
                }
            }
        }

        stage('Test HTTP Response') {
            steps {
                script {
                    // Проверяем, что Nginx отдает страницу
                    sh """
                    status_code=\$(curl -o /dev/null -s -w "%{http_code}" http://localhost:8080)
                    if [ "\$status_code" != "200" ]; then
                        echo "HTTP test failed with status \$status_code"
                        exit 1
                    fi
                    """
                }
            }
        }

        stage('Check MD5') {
            steps {
                script {
                    // Пример проверки MD5 файла index.html
                    sh """
                    md5sum index.html | awk '{print \$1}' > md5_local.txt
                    # Допустим, файл для проверки находится в контейнере
                    docker cp ${CONTAINER_NAME}:/usr/share/nginx/html/index.html index_container.html
                    md5sum index_container.html | awk '{print \$1}' > md5_container.txt
                    if ! diff md5_local.txt md5_container.txt >/dev/null; then
                        echo "MD5 mismatch!"
                        exit 1
                    fi
                    """
                }
            }
        }
    }

    post {
        success {
            script {
                sh """
                curl -s -X POST https://api.telegram.org/bot${TELEGRAM_TOKEN}/sendMessage \
                -d chat_id=${TELEGRAM_CHAT_ID} \
                -d text="✅ CI pipeline SUCCESS: ${JOB_NAME} #${BUILD_NUMBER}"
                """
                sh "docker rm -f ${CONTAINER_NAME} || true"
            }
        }
        failure {
            script {
                sh """
                curl -s -X POST https://api.telegram.org/bot${TELEGRAM_TOKEN}/sendMessage \
                -d chat_id=${TELEGRAM_CHAT_ID} \
                -d text="❌ CI pipeline FAILED: ${JOB_NAME} #${BUILD_NUMBER}"
                """
                sh "docker rm -f ${CONTAINER_NAME} || true"
            }
        }
        always {
            script {
                sh "docker rm -f ${CONTAINER_NAME} || true"
            }
        }
    }
}
