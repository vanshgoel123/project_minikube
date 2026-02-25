pipeline {
  agent any
  environment {
    DOCKERHUB_REPO = "vansh23100/devops-demo"
    IMAGE_TAG = "${env.BUILD_ID}"
  }
  stages {

    stage('Checkout Code') {
      steps {
        checkout scm
      }
    }
    stage('Build Docker Image') {
      steps {
        sh 'docker build --no-cache -t ${DOCKERHUB_REPO}:${IMAGE_TAG} .'
      }
    }
    stage('Push to Docker Hub') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
          sh 'docker push ${DOCKERHUB_REPO}:${IMAGE_TAG}'
        }
      }
    }
    stage('Deploy to Kubernetes') {
      steps {
        echo "Deploying to Kubernetes cluster..."
        sh '''
          export KUBECONFIG=/var/lib/jenkins/.kube/config
          kubectl config use-context minikube
          kubectl set image deployment/devops-demo devops-demo=${DOCKERHUB_REPO}:${IMAGE_TAG}
          kubectl rollout status deployment/devops-demo --timeout=5m
          echo "Deployment successful! Service running at: http://192.168.58.2:30080"
        '''
      }
    }
  }

  post {
    success {
      echo "CI/CD Pipeline Successful!"
    }
    failure {
      echo "Pipeline Failed!"
    }
  }
}
