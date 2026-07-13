pipeline {
    agent any

    triggers {
        githubPush()
    }

    environment {
        APP_DIR = 'Sucursal_vehiculos'
        IMAGE_NAME = 'vehiculos-app'
        CONTAINER_NAME = 'vehiculos-container'
        DOCKER_NETWORK = 'vehiculos-net'
    }

    stages {
        stage('Obtener código') {
            steps {
                checkout scm
            }
        }

        stage('Ejecutar pruebas') {
            steps {
                dir("${APP_DIR}") {
                    sh 'mvn -B test'
                }
            }
        }

        stage('Compilar aplicación') {
            steps {
                dir("${APP_DIR}") {
                    sh 'mvn -B clean package -DskipTests'
                }
            }
        }

        stage('Construir imagen Docker') {
            steps {
                dir("${APP_DIR}") {
                    sh 'docker build -t ${IMAGE_NAME}:latest .'
                }
            }
        }

        stage('Desplegar en Tomcat') {
            steps {
                sh '''
                    docker network inspect "$DOCKER_NETWORK" >/dev/null 2>&1 || \
                    docker network create "$DOCKER_NETWORK"

                    docker network connect "$DOCKER_NETWORK" mysql-container \
                    2>/dev/null || true

                    docker rm -f "$CONTAINER_NAME" 2>/dev/null || true

                    docker run -d \
                      --name "$CONTAINER_NAME" \
                      --restart unless-stopped \
                      --network "$DOCKER_NETWORK" \
                      -p 9090:8080 \
                      "$IMAGE_NAME:latest"
                '''
            }
        }

        stage('Validar despliegue') {
            steps {
                sh '''
                    sleep 20
                    docker ps --filter "name=$CONTAINER_NAME"
                    docker logs --tail 50 "$CONTAINER_NAME"
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline ejecutado correctamente.'
            echo 'Aplicación disponible en el puerto 9090.'
        }

        failure {
            echo 'El pipeline falló. Revisar la etapa marcada en rojo.'
        }

        always {
            junit testResults: 'Sucursal_vehiculos/target/surefire-reports/*.xml',
                  allowEmptyResults: true
        }
    }
}
