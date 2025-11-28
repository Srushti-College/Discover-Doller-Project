pipeline {
  agent any

  environment {
    DOCKERHUB_CREDENTIALS = 'dockerhub-creds'   
    DOCKERHUB_USER = 'srushti22'
    IMAGE_BACKEND = "srushti22/crud-backend"
    IMAGE_FRONTEND = "srushti22/crud-frontend"
    COMPOSE_PATH = "${WORKSPACE}"   
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build Backend Image') {
      steps {
        dir('backend') {
          script {
            docker.build("${env.IMAGE_BACKEND}:${env.BUILD_NUMBER}").push()
            docker.image("${env.IMAGE_BACKEND}:${env.BUILD_NUMBER}").push()
          }
        }
      }
    }

    stage('Build Frontend Image') {
      steps {
        dir('frontend') {
          script {
            docker.build("${env.IMAGE_FRONTEND}:${env.BUILD_NUMBER}").push()
            docker.image("${env.IMAGE_FRONTEND}:${env.BUILD_NUMBER}").push()
          }
        }
      }
    }

    stage('Tag latest') {
      steps {
        script {
          sh "docker login -u ${DOCKERHUB_USER} -p ${DOCKERHUB_PASSWORD:-placeholder}"
          sh "docker tag ${IMAGE_BACKEND}:${env.BUILD_NUMBER} ${IMAGE_BACKEND}:latest"
          sh "docker push ${IMAGE_BACKEND}:latest"
          sh "docker tag ${IMAGE_FRONTEND}:${env.BUILD_NUMBER} ${IMAGE_FRONTEND}:latest"
          sh "docker push ${IMAGE_FRONTEND}:latest"
        }
      }
    }

    stage('Deploy to Server') {
      steps {
        
        sh """
          
          docker run --rm -v /var/run/docker.sock:/var/run/docker.sock -v ${COMPOSE_PATH}:/app -w /app docker/compose:2.20.2 pull
          docker run --rm -v /var/run/docker.sock:/var/run/docker.sock -v ${COMPOSE_PATH}:/app -w /app docker/compose:2.20.2 up -d --remove-orphans
        """
      }
    }
  }

  post {
    success { echo "Pipeline completed" }
    failure { echo "Pipeline failed" }
  }
}

