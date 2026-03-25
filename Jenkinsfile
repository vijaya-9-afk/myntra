pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = "vijaya9494"       // Docker Hub credential ID
        IMAGE_NAME = "myntraimg"
        IMAGE_TAG = "${BUILD_NUMBER}"              // Build number as tag
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/vijaya-9-afk/myntra.git'
            }
        }

        stage('Build Artifact') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "${DOCKERHUB_CREDENTIALS}",
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker tag ${IMAGE_NAME}:${IMAGE_TAG} $DOCKER_USER/${IMAGE_NAME}:${IMAGE_TAG}
                        docker push $DOCKER_USER/${IMAGE_NAME}:${IMAGE_TAG}
                        docker logout
                    '''
                }
            }
        }

        stage('K8s Deployment') {
            steps {
                // Use your EKS kubeconfig stored as Jenkins credential
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh '''
                        echo "Using KUBECONFIG: $KUBECONFIG"
                        kubectl get nodes
                        
                        # Check workspace contents
                        ls -l

                        # Apply the Kubernetes manifest
                        kubectl apply -f myntra.yml
                    '''
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline finished."
        }
        failure {
            echo "Pipeline failed. Check logs."
            // Optional: send mail (configure SMTP first)
            // mail to: 'you@example.com',
            //      subject: "Jenkins Build Failed: ${JOB_NAME} #${BUILD_NUMBER}",
            //      body: "Check Jenkins console output: ${BUILD_URL}"
        }
    }
}
