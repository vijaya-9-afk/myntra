pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('vijaya9494')
        DOCKER_IMAGE = "myntraimg:latest"
        DOCKER_USER = "${DOCKERHUB_CREDENTIALS_USR}"
        AWS_DEFAULT_REGION = "ap-south-1"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/vijaya-9-afk/myntra.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Tag Docker Image') {
            steps {
                sh 'docker tag $DOCKER_IMAGE $DOCKER_USER/$DOCKER_IMAGE'
            }
        }

        stage('Push Docker Image') {
            steps {
                sh '''
                echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                docker push $DOCKER_USER/$DOCKER_IMAGE
                docker logout
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                aws eks update-kubeconfig --region $AWS_DEFAULT_REGION --name new-cluster1

                kubectl apply -f k8s/deployment.yml
                kubectl apply -f k8s/service.yml
                kubectl apply -f k8s/ingress.yml
                '''
            }
        }

        stage('Show App URL') {
            steps {
                sh '''
                echo "Waiting for Ingress..."

                for i in {1..18}; do
                    URL=$(kubectl get ingress k8s-ingress -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')

                    if [ ! -z "$URL" ]; then
                        echo "App URL: http://$URL/rose"
                        exit 0
                    fi

                    sleep 10
                done

                echo "LoadBalancer not ready"
                '''
            }
        }
    }

    post {
        success {
            emailext (
                subject: "✅ SUCCESS: Build #${BUILD_NUMBER}",
                body: """
                <h2>Build SUCCESS ✅</h2>
                <p><b>Job:</b> ${JOB_NAME}</p>
                <p><b>Build Number:</b> ${BUILD_NUMBER}</p>
                <p>Status: SUCCESS</p>
                <p>Check here: ${BUILD_URL}</p>
                """,
                to: "vijayakanthi9533@gmail.com"
            )
        }

        failure {
            emailext (
                subject: "❌ FAILED: Build #${BUILD_NUMBER}",
                body: """
                <h2>Build FAILED ❌</h2>
                <p><b>Job:</b> ${JOB_NAME}</p>
                <p><b>Build Number:</b> ${BUILD_NUMBER}</p>
                <p>Status: FAILURE</p>
                <p>Check logs: ${BUILD_URL}</p>
                """,
                to: "vijayakanthi9533@gmail.com"
            )
        }
    }
}
