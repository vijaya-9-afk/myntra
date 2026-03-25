pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "vijaya9494/myntraimg"
        DOCKER_TAG = "latest"
    }

    stages {

        stage('Checkout SCM') {
            steps {
                git url: 'https://github.com/vijaya-9-afk/myntra.git',
                    credentialsId: 'vijaya9494',
                    branch: 'main'
            }
        }

        stage('Build Artifact') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker_cred', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:${DOCKER_TAG}
                        docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                        docker logout
                    """
                }
            }
        }

        stage('K8s Deployment') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh "kubectl apply -f myntra.yml"
                }
            }
        }
    }

    post {
        success {
            mail to: 'your-email@example.com',
                 subject: "Jenkins Pipeline SUCCESS: ${env.JOB_NAME} [${env.BUILD_NUMBER}]",
                 body: "The pipeline completed successfully.\n\nCheck build details: ${env.BUILD_URL}"
        }
        failure {
            mail to: 'your-email@example.com',
                 subject: "Jenkins Pipeline FAILURE: ${env.JOB_NAME} [${env.BUILD_NUMBER}]",
                 body: "The pipeline failed.\n\nCheck build details: ${env.BUILD_URL}"
        }
    }
}
