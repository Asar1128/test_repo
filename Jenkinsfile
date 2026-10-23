pipeline {
    agent any

    environment {
        IMAGE_NAME = 'springboot-demo'
        CONTAINER   = 'springboot-demo'
        HOST_PORT   = '8081'   // app exposed on 8081 (Jenkins owns 8080)
        APP_PORT    = '8080'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} -t ${IMAGE_NAME}:latest .'
            }
        }

        stage('Stop Old Container') {
            steps {
                sh 'docker rm -f ${CONTAINER} || true'
            }
        }

        stage('Run New Container') {
            steps {
                sh '''
                    docker run -d --name ${CONTAINER} \
                        -p ${HOST_PORT}:${APP_PORT} \
                        ${IMAGE_NAME}:${BUILD_NUMBER}
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    for i in $(seq 1 30); do
                        curl -fs http://localhost:${HOST_PORT}/hello && exit 0
                        sleep 2
                    done
                    echo "Deployment verification failed"
                    docker logs ${CONTAINER}
                    exit 1
                '''
            }
        }
    }

    post {
        success {
            echo "Deployed ${IMAGE_NAME}:${BUILD_NUMBER} successfully"
        }
        failure {
            sh 'docker logs ${CONTAINER} || true'
        }
    }
}
