pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = "docker_cred"   // ✅ correct credential ID
        IMAGE_NAME = "myntraimg"
        IMAGE_TAG = "${BUILD_NUMBER}"           // ✅ better than latest
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

        stage('Build Image') {
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
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh '''
                        kubectl get nodes
                        kubectl apply -f myntra.yml
                    '''
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline execution completed"
        }

        success {
            mail to: 'devapulupureddy@gmail.com',
                 subject: "Jenkins SUCCESS: ${env.JOB_NAME}",
                 body: "Build succeeded: ${env.BUILD_URL}"
        }

        failure {
            mail to: 'devapulupureddy@gmail.com',
                 subject: "Jenkins FAILURE: ${env.JOB_NAME}",
                 body: "Build failed: ${env.BUILD_URL}"
        }
    }
}
