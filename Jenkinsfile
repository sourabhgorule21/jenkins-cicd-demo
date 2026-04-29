pipeline {
    agent any

    environment {
        IMAGE_NAME = 'jenkins-cicd-demo'
        APP_CONTAINER_NAME = 'jenkins-cicd-demo-app'
        MYSQL_CONTAINER_NAME = 'jenkins-cicd-demo-mysql'
        APP_NETWORK = 'jenkins-cicd-demo-net'
        DB_HOST = 'jenkins-cicd-demo-mysql'
        DB_PORT = '3306'
        DB_NAME = 'appdb'
        DB_USERNAME = 'appuser'
        DB_PASSWORD = 'apppassword'
        MYSQL_ROOT_PASSWORD = 'rootpassword'
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
                sh '''
                    docker rm -f ${APP_CONTAINER_NAME} || true
                    docker rm -f ${MYSQL_CONTAINER_NAME} || true
                    docker network rm ${APP_NETWORK} || true
                '''
            }
        }

        stage('Run new container') {
            steps {
                sh '''
                    docker network create ${APP_NETWORK}

                    docker run -d \
                      --name ${MYSQL_CONTAINER_NAME} \
                      --network ${APP_NETWORK} \
                      -e MYSQL_DATABASE=${DB_NAME} \
                      -e MYSQL_USER=${DB_USERNAME} \
                      -e MYSQL_PASSWORD=${DB_PASSWORD} \
                      -e MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD} \
                      --health-cmd="mysqladmin ping -h localhost -uroot -p${MYSQL_ROOT_PASSWORD}" \
                      --health-interval=10s \
                      --health-timeout=5s \
                      --health-retries=20 \
                      mysql:8.4

                    echo "Waiting for MySQL to become healthy..."
                    for i in $(seq 1 40); do
                      status=$(docker inspect --format='{{.State.Health.Status}}' ${MYSQL_CONTAINER_NAME})
                      if [ "$status" = "healthy" ]; then
                        break
                      fi
                      if [ "$status" = "unhealthy" ]; then
                        echo "MySQL is unhealthy"
                        docker logs ${MYSQL_CONTAINER_NAME}
                        exit 1
                      fi
                      sleep 3
                    done

                    docker run -d \
                      --name ${APP_CONTAINER_NAME} \
                      --network ${APP_NETWORK} \
                      -e DB_HOST=${DB_HOST} \
                      -e DB_PORT=${DB_PORT} \
                      -e DB_NAME=${DB_NAME} \
                      -e DB_USERNAME=${DB_USERNAME} \
                      -e DB_PASSWORD=${DB_PASSWORD} \
                      -p 9090:9090 \
                      ${IMAGE_NAME}:latest
                '''
            }
        }

        stage('Show logs') {
            steps {
                sh '''
                    docker logs --tail 200 ${MYSQL_CONTAINER_NAME} || true
                    docker logs --tail 200 ${APP_CONTAINER_NAME}
                '''
            }
        }
    }
}
