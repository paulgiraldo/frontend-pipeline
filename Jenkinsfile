pipeline {
    agent any

    parameters {
        string(name: 'BUCKET_FUENTE', defaultValue: 'bucket-codigo-backup', description: 'Nombre de bucket de origen...')
        string(name: 'CARPETA_USUARIO', defaultValue: 'paul', description: 'Nombre de la carpeta del Usuario..')

        choice(
            choices: ['AWS' , 'VERCEL'],
            description: '',
            name: 'DEPLOY_SERVER'
        )
        string(name: 'CARPETA_FUENTE', defaultValue: 'VERSION_1.1', description: 'Nombre de la carpeta del bucket  origen...')
        string(name: 'BUCKET_TARGET', defaultValue: 'bucket-codigo-paul', description: 'Nombre de bucket destino en caso "DEPLOY_SERVER = AWS" ...')
    }

    environment {
        WEBHOOK_URL = 'https://f23a-190-40-228-4.ngrok-free.app/github-webhook/' // URL del webhook
        AWS_REGION = 'us-east-1' // Región de AWS
        S3_BUCKET = 'bucket-codigo-paul' // Nombre del bucket S3
        RECIPIENT_EMAIL = 'paulgiraldo72@gmail.com' // Dirección de correo electrónico del destinatario
        BACKUP_BUCKET = 'bucket-codigo-backup' // Bucket para el respaldo

        VERCEL_TOKEN = credentials('VERCEL_TOKEN')

    }

    stages {
        stage('Mover archivos entre buckets s3 AWS ...') {
            when {
                expression { params.DEPLOY_SERVER == 'AWS' }
            }
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
                            aws s3 sync s3://${params.BUCKET_FUENTE}/${params.CARPETA_USUARIO}/master/${params.CARPETA_FUENTE}/ s3://${params.BUCKET_TARGET}/ --delete
                        """
                    }                   
                }
            }
        }
    

        stage('Mover archivos entre buckets s3 AWS hacia Carpeta build ...') {
            when {
                expression { params.DEPLOY_SERVER == 'VERCEL' }
            }
            agent {
                docker {
                    image 'amazon/aws-cli:2.23.7'
                    args '--entrypoint ""'
                }
            }
            steps {
                withAWS(credentials: 'aws-credentials-s3', region: 'us-east-1') {

                    script {
                      
                        echo "Sincronizando archivos entre buckets s3 y Vercel.."
                        sh "mkdir -p build"

                        sh """
                            aws s3 sync s3://${params.BUCKET_FUENTE}/${params.CARPETA_USUARIO}/vercel/${params.CARPETA_FUENTE}/ build/ --delete
                        """
                    }                   
                }
            }
        }
    

        stage('Mover archivos desde la Carpeta build hacia VERCEL...') {
            when {
                expression { params.DEPLOY_SERVER == 'VERCEL' }
            }
            agent {
                docker { image 'node:18-alpine'}
            }
            steps {
                sh """
                    npm install -g vercel
                    vercel deploy --prod --name front-vercel --token $VERCEL_TOKEN --yes
                """
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