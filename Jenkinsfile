pipeline {
    agent any

    environment {
        DOCKERHUB_CREDS = credentials('dockerhub_creds')
        KUBECONFIG_FILE = credentials('kubeconfig')
        MYSQL_IMAGE = "pavdev777/mysql-demo"
        PHPMYADMIN_IMAGE = "pavdev777/mysql-demo-phpmyadmin"
        TAG = "latest"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Images') {
            steps {
                sh "docker build -t ${MYSQL_IMAGE}:${TAG} -f Dockerfile.mysql ."
                sh "docker build -t ${PHPMYADMIN_IMAGE}:${TAG} -f Dockerfile.phpmyadmin ."
            }
        }

        stage('Push Images') {
            steps {
                sh "echo ${DOCKERHUB_CREDS_PSW} | docker login -u ${DOCKERHUB_CREDS_USR} --password-stdin"
                sh "docker push ${MYSQL_IMAGE}:${TAG}"
                sh "docker push ${PHPMYADMIN_IMAGE}:${TAG}"
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withFile([file: KUBECONFIG_FILE, variable: 'KUBECONFIG']) {
                    sh "kubectl apply -f k8s/mysql-deployment.yaml"
                    sh "kubectl apply -f k8s/mysql-service.yaml"
                    sh "kubectl apply -f k8s/phpmyadmin-deployment.yaml"
                    sh "kubectl apply -f k8s/phpmyadmin-service.yaml"
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline finished"
        }
    }
}