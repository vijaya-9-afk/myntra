pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "myntraimg:latest"
        KUBE_MANIFEST = "myntra.yml"
        SMTP_SERVER = "smtp.gmail.com"        // Change to your SMTP host
        SMTP_PORT = "587"                     // Usually 587 for TLS
        EMAIL_FROM = "vijayakanthi9533@gmail.com"
        EMAIL_TO = "recipient@example.com"
        EMAIL_CRED = "email-creds"            // Jenkins credential ID for email (username/password)
    }

    stages {
        stage('Checkout SCM') {
            steps {
                git url: 'https://github.com/vijaya-9-afk/myntra.git', branch: 'main', credentialsId: 'vijaya9494'
            }
        }

        stage('Build Artifact') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE} ."
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    try {
                        withCredentials([usernamePassword(credentialsId: 'docker-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                            sh '''
                                echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                                docker tag ${DOCKER_IMAGE} $DOCKER_USER/${DOCKER_IMAGE}
                                docker push $DOCKER_USER/${DOCKER_IMAGE}
                                docker logout
                            '''
                        }
                    } catch (Exception e) {
                        echo "Docker push failed: ${e}"
                        error("Stopping pipeline due to Docker push failure")
                    }
                }
            }
        }

        stage('K8s Deployment') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh '''
                        export KUBECONFIG=$KUBECONFIG
                        kubectl get nodes
                        kubectl apply -f ${KUBE_MANIFEST}
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline Succeeded! Sending email...'
            script {
                withCredentials([usernamePassword(credentialsId: EMAIL_CRED, usernameVariable: 'MAIL_USER', passwordVariable: 'MAIL_PASS')]) {
                    mail bcc: '',
                         body: 'The Jenkins pipeline has succeeded!',
                         cc: '',
                         from: EMAIL_FROM,
                         replyTo: EMAIL_FROM,
                         subject: 'Pipeline Success',
                         to: EMAIL_TO,
                         mimeType: 'text/plain',
                         smtpHost: SMTP_SERVER,
                         smtpPort: SMTP_PORT,
                         smtpAuthUser: MAIL_USER,
                         smtpAuthPassword: MAIL_PASS,
                         useSsl: false,
                         useTls: true
                }
            }
        }

        failure {
            echo 'Pipeline Failed! Sending email...'
            script {
                withCredentials([usernamePassword(credentialsId: EMAIL_CRED, usernameVariable: 'MAIL_USER', passwordVariable: 'MAIL_PASS')]) {
                    mail bcc: '',
                         body: 'The Jenkins pipeline has failed!',
                         cc: '',
                         from: EMAIL_FROM,
                         replyTo: EMAIL_FROM,
                         subject: 'Pipeline Failure',
                         to: EMAIL_TO,
                         mimeType: 'text/plain',
                         smtpHost: SMTP_SERVER,
                         smtpPort: SMTP_PORT,
                         smtpAuthUser: MAIL_USER,
                         smtpAuthPassword: MAIL_PASS,
                         useSsl: false,
                         useTls: true
                }
            }
        }
    }
}
