pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = "vijaya9494"
        IMAGE_NAME = "myntraimg"
        IMAGE_TAG = "${BUILD_NUMBER}"       // safer than "latest"
        EMAIL_RECIPIENTS = "vijayakanti9533@gmail.com"
        KUBECONFIG_CRED = "kubeconfig"
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
                    passwordVariable: 'DOCKER_PASS')]) {
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
                withCredentials([file(credentialsId: "${KUBECONFIG_CRED}", variable: 'KUBECONFIG')]) {
                    sh '''
                        export KUBECONFIG=$KUBECONFIG
                        kubectl get nodes

                        # Dynamically find manifest
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
            echo "Pipeline Succeeded! Sending email..."
            mail to: "${EMAIL_RECIPIENTS}",
                 subject: "SUCCESS: Jenkins Pipeline ${JOB_NAME} Build #${BUILD_NUMBER}",
                 body: "Good news! The pipeline ${JOB_NAME} Build #${BUILD_NUMBER} succeeded.\n\nCheck details: ${BUILD_URL}"
        }

        failure {
            echo "Pipeline Failed! Sending email..."
            mail to: "${EMAIL_RECIPIENTS}",
                 subject: "FAILURE: Jenkins Pipeline ${JOB_NAME} Build #${BUILD_NUMBER}",
                 body: "Oops! The pipeline ${JOB_NAME} Build #${BUILD_NUMBER} failed.\n\nCheck details: ${BUILD_URL}"
        }
    }
}
