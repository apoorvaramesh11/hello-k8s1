pipeline {
    agent any

    environment {
     
        IMAGE_NAME = "YOUR_DOCKERHUB_USERNAME/hello-k8s"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }


        stage('Deploy to kind') {
            steps {
                sh """
                    
                    kind load docker-image ${IMAGE_NAME}:${IMAGE_TAG}
                    sed -i 's|${IMAGE_NAME}:latest|${IMAGE_NAME}:${IMAGE_TAG}|g' k8s/deployment.yaml

                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml

                    

                    kubectl port-forward svc/hello-k8s 5005:5005 &
                """
            }
        }

    }

    post {
        success {
            echo "✅ Deployed successfully — http://localhost:5005"
        }
        failure {
            echo "❌ Pipeline failed. Check logs above."
        }
    }
}
