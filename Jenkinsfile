pipeline {
  agent any

  environment {
    IMAGE_NAME = "myapp"
    IMAGE_TAG = "latest"
  }

  stages {
    stage('Checkout') {
      steps {
        // клонируем код из SCM
        checkout scm
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          sh """
            echo "Building Docker image..."
            docker build -t $IMAGE_NAME:$IMAGE_TAG .
          """
        }
      }
    }

    stage('Test Container') {
      steps {
        script {
          sh """
            echo "Running container test..."
            docker run --rm -d -p 3000:3000 --name test_app $IMAGE_NAME:$IMAGE_TAG
            sleep 5
            curl -f http://localhost:3000 || (echo 'App did not respond' && exit 1)
            docker stop test_app
          """
        }
      }
    }

    stage('Push to Registry (optional)') {
      when { expression { return false } } // отключено пока
      steps {
        script {
          sh """
            echo "Login to registry..."
            docker login -u \$DOCKER_USER -p \$DOCKER_PASS
            docker tag $IMAGE_NAME:$IMAGE_TAG my-dockerhub-user/$IMAGE_NAME:$IMAGE_TAG
            docker push my-dockerhub-user/$IMAGE_NAME:$IMAGE_TAG
          """
        }
      }
    }
  }

  post {
    success {
      echo "✅ Build & test completed successfully!"
    }
    failure {
      echo "❌ Build failed!"
    }
  }
}