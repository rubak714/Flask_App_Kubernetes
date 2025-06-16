pipeline {
  agent any

  environment {
    IMAGE_NAME = "flask-cicd-app"
    CONTAINER_NAME = "flask_app"
    DOCKERFILE_DIR = "/home/rubaiya/Desktop/devops_github/Flask_App_Kubernetes"
  }

  stages {
    stage('Build Docker Image') {
      steps {
        script {
          sh 'docker version' // Ensure Docker is accessible
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
          sh 'kubectl apply -f /home/rubaiya/Desktop/devops_github/Flask_App_Kubernetes/k8s/deployment.yaml'
          sh 'kubectl apply -f /home/rubaiya/Desktop/devops_github/Flask_App_Kubernetes/k8s/service.yaml'
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
