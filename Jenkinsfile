pipeline {
    agent any

    environment {
        IMAGE_NAME = "myapp"
        IMAGE_TAG = "latest"
        KUBE_NAMESPACE = "default"
        KUBECONFIG = "/var/lib/jenkins/.kube/config"
        MINIKUBE_HOME = "/var/lib/jenkins/.minikube"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

    stage('Build Docker Image') {
        steps {
            sh """
                echo "=== Building Docker image ==="
                export DOCKER_TLS_VERIFY="1"
                export DOCKER_HOST="tcp://192.168.49.2:2376"
                export DOCKER_CERT_PATH="/home/pavel/.minikube/certs"
                export MINIKUBE_ACTIVE_DOCKERD="minikube"
                docker build -t $IMAGE_NAME:$IMAGE_TAG .
                docker images
            """
        }
    }

        stage('Test Container') {
            steps {
                sh """
                    echo "=== Testing container ==="
                    docker run --rm -d -p 3000:3000 --name test_app $IMAGE_NAME:$IMAGE_TAG
                    sleep 5
                    curl -f http://localhost:3000 || (echo 'App did not respond' && exit 1)
                    docker stop test_app
                """
            }
        }

        stage('Deploy to Minikube') {
            steps {
                sh """
                    echo "=== Deploying to minikube ==="
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
                    kubectl get pods -n $KUBE_NAMESPACE
                """
            }
        }
    }

    post {
        success { echo "✅ Pipeline finished successfully!" }
        failure { echo "❌ Pipeline failed!" }
    }
}
