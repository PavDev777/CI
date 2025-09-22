pipeline {
  agent any

  environment {
    MINIKUBE_HOME = "/var/lib/jenkins/.minikube"
    KUBECONFIG = "/var/lib/jenkins/.kube/config"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build MySQL Image') {
      steps {
        sh '''
          export MINIKUBE_HOME=${MINIKUBE_HOME}
          eval $(minikube docker-env --shell bash)
          docker build -t mysql:latest ./mysql
        '''
      }
    }

    stage('Build phpMyAdmin Image') {
      steps {
        sh '''
          export MINIKUBE_HOME=${MINIKUBE_HOME}
          eval $(minikube docker-env --shell bash)
          docker build -t phpmyadmin:latest ./phpmyadmin
        '''
      }
    }

    stage('Deploy to Minikube') {
      steps {
        sh '''
          export KUBECONFIG=${KUBECONFIG}
          kubectl apply -f k8s/mysql-deployment.yaml
          kubectl apply -f k8s/mysql-service.yaml
          kubectl apply -f k8s/phpmyadmin-deployment.yaml
          kubectl apply -f k8s/phpmyadmin-service.yaml
          kubectl rollout status deployment/mysql
          kubectl rollout status deployment/phpmyadmin
        '''
      }
    }
  }

  post {
    success {
      echo "✅ CI/CD pipeline finished successfully!"
    }
    failure {
      echo "❌ Pipeline failed!"
    }
  }
}