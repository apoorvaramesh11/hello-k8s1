pipeline {
    agent any

    environment {
        IMAGE_NAME = "YOUR_DOCKERHUB_USERNAME/hello-k8s"
        IMAGE_TAG = "${BUILD_NUMBER}"
        DOCKER_CREDENTIALS_ID = "dockerhub-creds"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        
        
        stage('Push Docker Image') {
            steps {
                sh """
                    docker push ${IMAGE_NAME}:${IMAGE_TAG}
                    docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest
                    docker push ${IMAGE_NAME}:latest
                """
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh """
                    kubectl apply -f deployment.yaml
                    kubectl apply -f service.yaml

                    kubectl set image deployment/hello-k8s \
                    hello-k8s=${IMAGE_NAME}:${IMAGE_TAG}

                    kubectl rollout status deployment/hello-k8s
                """
            }
        }

        stage('Verify Deployment') {
            steps {
                sh "kubectl get pods -o wide"
                sh "kubectl get svc"
            }
        }
    }

    post {
        success {
            echo "✅ Deployment successful!"
            echo "🌐 Access your app using NodePort or port-forward:"
            echo "kubectl port-forward svc/hello-k8s 5005:5005"
        }

        failure {
            echo "❌ Pipeline failed. Check logs above."
        }

        always {
            sh "docker logout || true"
        }
    }
}
