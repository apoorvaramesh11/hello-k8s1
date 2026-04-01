pipeline {
    agent any

    environment {
        IMAGE_NAME = "apoorvar12/hello-k8s"
        IMAGE_TAG = "${BUILD_NUMBER}"
        
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/apoorvaramesh11/hello-k8s1.git'
    
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Login to Docker Hub') {
    steps {
        withCredentials([usernamePassword(
            credentialsId: 'DOCKERHUB_ID',
            usernameVariable: 'DOCKER_USER',
            passwordVariable: 'DOCKER_PASS'
        )]) {
            sh """
                echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
            """
        }
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
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml

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
