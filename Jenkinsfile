pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "myntraimg:latest"
        KUBE_MANIFEST = "myntra.yml"
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
                        withCredentials([usernamePassword(
                            credentialsId: 'docker-creds',
                            usernameVariable: 'DOCKER_USER',
                            passwordVariable: 'DOCKER_PASS'
                        )]) {
                            sh '''
                                echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                                docker tag ${DOCKER_IMAGE} $DOCKER_USER/${DOCKER_IMAGE}
                                docker push $DOCKER_USER/${DOCKER_IMAGE}
                                docker logout
                            '''
                        }
                    } catch (e) {
                        echo "Docker credentials missing! Skipping Docker push."
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
                        echo "Kubernetes deployment failed."
                    }
                }
            }
        }
    }

    post {
        always {
            emailext(
                subject: "Jenkins Build: ${currentBuild.currentResult}",
                body: """Build Status: ${currentBuild.currentResult}

Job: ${env.JOB_NAME}
Build URL: ${env.BUILD_URL}
""",
                to: "${EMAIL_TO}"
            )
        }
    }
}
