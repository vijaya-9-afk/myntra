pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "myntraimg:latest"
        KUBE_MANIFEST = "myntra.yml"
        EMAIL_FROM = "vijayakanthi9533@gmail.com"
        EMAIL_TO = "vijayakanthi9533@gmail.com"
    }

    stages {
        stage('Checkout SCM') {
            steps {
                git url: 'https://github.com/vijaya-9-afk/myntra.git', 
                    branch: 'main', 
                    credentialsId: 'vijaya9494'
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
                        withCredentials([usernamePassword(credentialsId: 'docker-creds', 
                                                         usernameVariable: 'DOCKER_USER', 
                                                         passwordVariable: 'DOCKER_PASS')]) {
                            sh '''
                                echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                                docker tag ${DOCKER_IMAGE} $DOCKER_USER/${DOCKER_IMAGE}
                                docker push $DOCKER_USER/${DOCKER_IMAGE}
                                docker logout
                            '''
                        }
                    } catch (Exception e) {
                        echo "Docker credentials missing or login failed! Skipping Docker push."
                        echo "Error details: ${e}"
                    }
                }
            }
        }

        stage('K8s Deployment') {
            steps {
                script {
                    try {
                        withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                            sh '''
                                export KUBECONFIG=$KUBECONFIG
                                kubectl get nodes
                                kubectl apply -f ${KUBE_MANIFEST}
                            '''
                        }
                    } catch (Exception e) {
                        echo "Kubernetes credentials missing or deployment failed! Skipping K8s deployment."
                        echo "Error details: ${e}"
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline Succeeded! Sending email...'
            script {
                try {
                    withCredentials([usernamePassword(credentialsId: 'email-creds', 
                                                     usernameVariable: 'MAIL_USER', 
                                                     passwordVariable: 'MAIL_PASS')]) {
                        mail to: EMAIL_TO,
                             from: EMAIL_FROM,
                             replyTo: EMAIL_FROM,
                             subject: 'Pipeline Success',
                             body: 'The Jenkins pipeline has succeeded!',
                             mimeType: 'text/plain',
                             smtpHost: 'smtp.gmail.com',
                             smtpPort: '587',
                             smtpAuthUser: MAIL_USER,
                             smtpAuthPassword: MAIL_PASS,
                             useTls: true
                    }
                } catch (Exception e) {
                    echo "Email credentials missing! Skipping success email."
                    echo "Error details: ${e}"
                }
            }
        }

        failure {
            echo 'Pipeline Failed! Sending email...'
            script {
                try {
                    withCredentials([usernamePassword(credentialsId: 'email-creds', 
                                                     usernameVariable: 'MAIL_USER', 
                                                     passwordVariable: 'MAIL_PASS')]) {
                        mail to: EMAIL_TO,
                             from: EMAIL_FROM,
                             replyTo: EMAIL_FROM,
                             subject: 'Pipeline Failure',
                             body: 'The Jenkins pipeline has failed!',
                             mimeType: 'text/plain',
                             smtpHost: 'smtp.gmail.com',
                             smtpPort: '587',
                             smtpAuthUser: MAIL_USER,
                             smtpAuthPassword: MAIL_PASS,
                             useTls: true
                    }
                } catch (Exception e) {
                    echo "Email credentials missing! Skipping failure email."
                    echo "Error details: ${e}"
                }
            }
        }
    }
}
