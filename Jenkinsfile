pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = "vijaya9494"   // DockerHub credentials ID
        IMAGE_NAME = "myntraimg"
        IMAGE_TAG = "${BUILD_NUMBER}"           // Use build number for tagging
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
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh '''
                        echo "Using KUBECONFIG: $KUBECONFIG"
                        kubectl get nodes

                        # Find the manifest dynamically
                        MANIFEST=$(find . -name "myntra.yml" | head -n 1)
                        if [ -z "$MANIFEST" ]; then
                            echo "Error: myntra.yml not found!"
                            exit 1
                        fi

                        echo "Deploying using manifest: $MANIFEST"
                        kubectl apply -f $MANIFEST
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed! Check logs for details."
        }
    }
}
