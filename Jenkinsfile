pipeline {
    agent any

    environment {
        IMAGE_NAME = 'jenkins-cicd-demo'
        CONTAINER_NAME = 'jenkins-cicd-demo-app'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker image') {
            steps {
                sh '''
                    docker build \
                      -t ${IMAGE_NAME}:${BUILD_NUMBER} \
                      -t ${IMAGE_NAME}:latest \
                      .
                '''
            }
        }

        stage('Stop old container (if exists)') {
            steps {
                sh 'docker rm -f ${CONTAINER_NAME} || true'
            }
        }

        stage('Run new container') {
            steps {
                sh '''
                    docker run -d \
                      --name ${CONTAINER_NAME} \
                      -p 9090:9090 \
                      ${IMAGE_NAME}:latest
                '''
            }
        }

        stage('Show logs') {
            steps {
                sh 'docker logs --tail 200 ${CONTAINER_NAME}'
            }
        }
    }
}
