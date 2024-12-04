pipeline {
    agent any
    tools {
        nodejs 'node'
    }
    environment {
        IMAGE_NAME = 'nodejs-helloworld-api'
        CONTAINER_NAME = 'helloworld-container'
    }
    
    // Estaciones de ejecución del pipeline
    stages {
        // integración contínua.
        stage('Instalar dependencias') {
            steps {
                sh 'npm install'
            }
        }
        stage('Ejecutar pruebas') {
            steps {
                sh 'npm test'
            }
        }
        // Despliegue continuo.
        stage('Construir la imágen de Docker') {
            steps {
                // Construir la imagen de Docker
                sh "docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} ."
            }
        }
        
        stage('Stop and Remove Container') {
           steps {
               // Detener y eliminar el contenedor
                sh "docker stop ${CONTAINER_NAME} || true"
               sh "docker rm -f ${CONTAINER_NAME} || true"
            }
        }
        
        stage('Run Docker Container') {
            steps {
                // Ejecutar el contenedor
                sh "docker stop ${env.CONTAINER_NAME} || true"
                sh "docker rm -f ${env.CONTAINER_NAME} || true"
                sh "docker run -d -p 3000:3000 --name ${env.CONTAINER_NAME} ${IMAGE_NAME}:${BUILD_NUMBER}"
            }
        }
        
        stage('Test Application') {
            steps {
                script {
                    // Esperar un momento para que la aplicación esté lista
                    sleep(time: 5, unit: 'SECONDS')
                    // Realizar una solicitud CURL para verificar la respuesta
                    def curlOutput = sh(script: "curl -s -o /dev/null -w '%{http_code}' http://localhost:3000/", returnStdout: true).trim()
                    echo "curlOutput: ${curlOutput}"

                    // Verificar si el código de respuesta es 200
                    if (curlOutput == '200') {
                        currentBuild.result = 'SUCCESS'
                    } else {
                        currentBuild.result = 'FAILURE'
                        error("El código de respuesta no es 200, en su lugar es ${curlOutput}")
                    }
                }
            }
        }

    }
}
