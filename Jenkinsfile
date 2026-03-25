pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "myntraimg:latest"
        KUBE_MANIFEST = "myntra.yml"
        Email_details = "gmail-creds"
    }

    stages {
        stage('Checkout SCM') {
            steps {
                git url: 'https://github.com/vijaya-9-afk/myntra.git',
                    branch: 'main',
                    credentialsId: 'vijaya9494'
            }
             post {
                success {
                    emailext(
                        subject: "Checkout Success",
                        body: "Checkout completed",
                        to: "${Email_details}"
                    )
                }
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
                    } catch (e) {
                        echo "Docker credentials missing or login failed! Skipping Docker push."
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
                    } catch (e) {
                        echo "Kubernetes deployment failed or kubeconfig missing."
                    }
                }
            }
        }
    }
}
