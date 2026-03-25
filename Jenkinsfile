pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "myntraimg:latest"
        KUBE_MANIFEST = "myntra.yml"
    }

    stages {
        stage('Checkout SCM') {
            steps {
                git url: 'https://github.com/vijaya-9-afk/myntra.git', branch: 'main', credentialsId: 'vijaya9494'
            }
        }

        stage('Create Artifact') {
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
                withCredentials([usernamePassword(credentialsId: 'docker-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker tag ${DOCKER_IMAGE} $DOCKER_USER/${DOCKER_IMAGE}
                        docker push $DOCKER_USER/${DOCKER_IMAGE}
                        docker logout
                    '''
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
            withCredentials([usernamePassword(credentialsId: 'gmail-creds', usernameVariable: 'SMTP_USER', passwordVariable: 'SMTP_PASS')]) {
                mail to: 'recipient@example.com',
                     subject: 'Pipeline Success',
                     body: 'The Jenkins pipeline has succeeded!',
                     from: 'vijayakanthi9533@gmail.com',
                     replyTo: 'vijayakanthi9533@gmail.com',
                     smtpHost: 'smtp.gmail.com',
                     smtpPort: '587',
                     smtpUsername: SMTP_USER,
                     smtpPassword: SMTP_PASS,
                     smtpTls: true
            }
        }

        failure {
            echo 'Pipeline Failed! Sending email...'
            withCredentials([usernamePassword(credentialsId: 'gmail-creds', usernameVariable: 'SMTP_USER', passwordVariable: 'SMTP_PASS')]) {
                mail to: 'recipient@example.com',
                     subject: 'Pipeline Failure',
                     body: 'The Jenkins pipeline has failed!',
                     from: 'vijayakanthi9533@gmail.com',
                     replyTo: 'vijayakanthi9533@gmail.com',
                     smtpHost: 'smtp.gmail.com',
                     smtpPort: '587',
                     smtpUsername: SMTP_USER,
                     smtpPassword: SMTP_PASS,
                     smtpTls: true
            }
        }
    }
}
