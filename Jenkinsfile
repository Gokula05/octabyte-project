pipeline {
  agent any

  environment {
    DOCKER_IMAGE = "gokula05/octabyte-nginx"
    REGISTRY_CREDENTIALS = credentials('dockerhub-creds')
    DEPLOY_SERVER = "3.110.188.114"
    SSH_KEY = credentials('ec2-ssh-key')
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/Gokula05/octabyte-project.git'
      }
    }

    stage('Unit Tests') {
      steps {
        echo 'Run unit tests here'
      }
    }

    stage('Vulnerability Scan') {
      steps {
        echo 'Running npm audit'
        sh 'npm audit || true'
      }
    }

    stage('Docker Build') {
      steps {
        sh "docker build -t $DOCKER_IMAGE ."
      }
    }

    stage('Push to DockerHub') {
      steps {
        withDockerRegistry([credentialsId: 'dockerhub-creds']) {
          sh "docker push $DOCKER_IMAGE"
        }
      }
    }

    stage('Deploy to Staging') {
      steps {
        sshagent (credentials: ['ec2-ssh-key']) {
          sh """
            ssh -o StrictHostKeyChecking=no ec2-user@$DEPLOY_SERVER '
              docker rm -f web || true
              docker pull $DOCKER_IMAGE
              docker run -d -p 80:80 --name web $DOCKER_IMAGE
            '
          """
        }
      }
    }

    stage('Manual Approval') {
      steps {
        input message: 'Approve to deploy to production?'
      }
    }

    stage('Deploy to Production') {
      steps {
        echo 'Deployment to production goes here.'
      }
    }
  }

  post {
    failure {
      echo 'Build Failed!'
    }
  }
}


