pipeline {
    agent any

    environment {
        IMAGE_NAME = "eval3_app:latest"
        APP_CONTAINER = "eval3_app_container"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Image') {
            steps {
                sh 'docker build -t ${IMAGE_NAME} .'
            }
        }

        stage('Run App Container') {
            steps {
                sh '''
                docker rm -f ${APP_CONTAINER} || true
                docker run -d --name ${APP_CONTAINER} -p 5000:5000 ${IMAGE_NAME}
                sleep 5
                '''
            }
        }

        stage('OWASP ZAP Scan') {
            steps {
                sh """
                docker run --rm \
                    --user root \
                    -w /zap/wrk \
                    -v ${WORKSPACE}:/zap/wrk \
                    ghcr.io/zaproxy/zaproxy:stable \
                    zap-baseline.py \
                    -t http://host.docker.internal:5000 \
                    -j zap_report.json || true

                    # covertir JSON a HTML

                    docker run -- rm \
                        -v ${WORKSPACE}:/zap/wrk \
                        ghcr.io/zaproxy/zaproxy:stable \
                        zap-cli report -f html -o zap_report.html -i zap_report.json
                """
            }
        }

        stage('Archive Report') {
            steps {
                archiveArtifacts artifacts: 'zap_report.html', fingerprint: true
            }
        }
    }

    post {
        always {
            sh 'docker rm -f ${APP_CONTAINER} || true'
        }
    }
}
