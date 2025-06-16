pipeline {
  agent any

  environment {
    IMAGE_NAME = "flask-cicd-app"
    CONTAINER_NAME = "flask_app"
    KUBECONFIG_PATH = "/var/jenkins_home/.kube/config"
  }

  stages {
    stage('Build Docker Image') {
      steps {
        script {
          sh 'docker version' // Confirm Docker is accessible
          sh "docker build -t $IMAGE_NAME ." // Builds from the Jenkins workspace root
        }
      }
    }

    stage('Stop Existing Container') {
      steps {
        script {
          sh '''
            if [ "$(docker ps -aq -f name=^flask_app$)" ]; then
              echo "Removing existing container..."
              docker rm -f flask_app
            else
              echo "No existing container found."
            fi
          '''
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
          sh "kubectl --kubeconfig=$KUBECONFIG_PATH apply -f k8s/deployment.yaml"
          sh "kubectl --kubeconfig=$KUBECONFIG_PATH apply -f k8s/service.yaml"
        }
      }
    }

    stage('Health Check and Rollback') {
      steps {
        script {
          def status = sh(script: "kubectl --kubeconfig=$KUBECONFIG_PATH get pods | grep flask-app | grep Running | wc -l", returnStdout: true).trim()
          if (status != '1') {
            echo 'Deployment failed. Rolling back...'
            sh "kubectl --kubeconfig=$KUBECONFIG_PATH rollout undo deployment/flask-app-deployment"
          } else {
            echo 'Deployment healthy.'
          }
        }
      }
    }

    stage('Debug Containers') {
      steps {
        script {
          sh 'docker ps -a'
        }
      }
    }
  }

  post {
    success {
      echo '✅ Pipeline completed!'
    }
    failure {
      echo '❌ Pipeline failed!'
    }
  }
}
