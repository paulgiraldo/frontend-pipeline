pipeline {
    agent any

    parameters {
        string(name: 'BUCKET_FUENTE', defaultValue: 'bucket-codigo-backup', description: 'Nombre de bucket de origen...')
        string(name: 'BUCKET_TARGET', defaultValue: 'bucket-codigo-paul', description: 'Nombre de bucket objetivo...')
        string(name: 'CARPETA_USUARIO', defaultValue: 'paul', description: 'Nombre de la carpeta del Usuario..')
        string(name: 'CARPETA_FUENTE', defaultValue: 'VERSION_1.1', description: 'Nombre de la carpeta del bucket  origen...')
    }

    environment {
        WEBHOOK_URL = 'https://f23a-190-40-228-4.ngrok-free.app/github-webhook/' // URL del webhook
        AWS_REGION = 'us-east-1' // Región de AWS
        S3_BUCKET = 'bucket-codigo-paul' // Nombre del bucket S3
        RECIPIENT_EMAIL = 'paulgiraldo72@gmail.com' // Dirección de correo electrónico del destinatario
        BACKUP_BUCKET = 'bucket-codigo-backup' // Bucket para el respaldo
    }

    stages {
        stage('Validar parametros...') {
            agent any
            steps {
                script {
                    echo "BUCKET DE ORIGEN: ${params.BUCKET_FUENTE}"
                    echo "BUCKET DE DESTINO: ${params.BUCKET_TARGET}"
                    echo "CARPETA DE USUARIO: ${params.CARPETA_USUARIO}"
                    echo "CARPETA DE ORIGEN: ${params.CARPETA_FUENTE}"
                }
            }
        }

        stage('Mover archivos entre buckets s3 AWS ...') {
            agent {
                docker {
                    image 'amazon/aws-cli:2.23.7'
                    args '--entrypoint ""'
                }
            }
            steps {
                withAWS(credentials: 'aws-credentials-s3', region: 'us-east-1') {

                    script {
                        echo "Limpiando bucket objetivo..."

                        sh """
                            aws s3 rm s3://${params.BUCKET_TARGET}/ --recursive
                        """
                      
                        echo "Sincronizando archivos entre buckets s3..."
                        sh """
                            aws s3 sync s3://${params.BUCKET_FUENTE}/${params.CARPETA_USUARIO}/${params.CARPETA_FUENTE}/ s3://${params.BUCKET_TARGET}/ --delete
                        """
                    }                   
                }
            }
        }
    }

    post {
        success {
            mail to: 'paulgiraldo72@gmail.com',
                subject: "Pipeline ${env.JOB_NAME} ejecucion correcta",
                body: """
                Hola,

                El pipeline '${env.JOB_NAME}' (Build #${env.BUILD_NUMBER}) ha finalizado de manera correcta.

                Los detalles se pueden revisar en el siguiente enlace:
                ${env.BUILD_URL}

                Saludos,
                Jenkins Server
                """
        }
        failure {
            mail to: 'paulgiraldo72@gmail.com',
                subject: "⚠️ Pipeline ${env.JOB_NAME} falló",
                body: """
                Hola,

                El pipeline '${env.JOB_NAME}' (Build #${env.BUILD_NUMBER}) ha fallado.

                Revisa los detalles del error en el siguiente enlace:
                ${env.BUILD_URL}

                Saludos,
                Jenkins Server
                """
        }
    }
}