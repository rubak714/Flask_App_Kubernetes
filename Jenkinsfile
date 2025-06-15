pipeline {
  agent any

  environment {
    IMAGE_NAME = "flask-cicd-app"
    CONTAINER_NAME = "flask_app"
  }

  stages {
    stage('Build Docker Image') {
      steps {
        script {
          sh 'docker version' // 👉 Ensure Docker CLI is accessible
          sh "docker build -t $IMAGE_NAME ."
        }
      }
    }

    stage('Stop Existing Container') {
      steps {
        script {
          sh "docker ps -q --filter name=$CONTAINER_NAME | grep -q . && docker rm -f $CONTAINER_NAME || true"
        }
      }
    }

    stage('Run Docker Container') {
      steps {
        script {
          sh "docker run -d -p 5000:5000 --name $CONTAINER_NAME $IMAGE_NAME"
        }
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        script {
          sh 'kubectl apply -f deployment.yaml'
          sh 'kubectl apply -f k8s/service.yaml'
        }
      }
    }

    stage('Health Check and Rollback') {
      steps {
        script {
          def status = sh(script: "kubectl get pods | grep flask-app | grep Running | wc -l", returnStdout: true).trim()
          if (status != '1') {
            echo 'Deployment failed. Rolling back...'
            sh 'kubectl rollout undo deployment/flask-app-deployment'
          } else {
            echo 'Deployment healthy.'
          }
        }
      }
    }
  }

  post {
    success {
      echo 'Pipeline completed!'
    }
    failure {
      echo 'Pipeline failed!'
    }
  }
}
