pipeline {
    agent { label 'docker' }   
    ...
 }

  environment {
    DOCKERHUB_CREDENTIALS = 'dockerhub-creds'    
    IMAGE_BACKEND = "srushti22/crud-backend"
    IMAGE_FRONTEND = "srushti22/crud-frontend"
    COMPOSE_PATH = "${env.WORKSPACE}"            
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Verify docker CLI') {
      steps {
        sh '''
          echo "Checking docker availability..."
          if ! command -v docker >/dev/null 2>&1; then
            echo "ERROR: docker CLI not found on this agent."
            echo "If Jenkins runs in a container, restart it mapping /var/run/docker.sock:/var/run/docker.sock and /usr/bin/docker:/usr/bin/docker"
            exit 127
          fi
          docker --version || true
        '''
      }
    }

    stage('Login to Docker Hub') {
      steps {
        
        withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIALS}",
                                          usernameVariable: 'DOCKERHUB_USER',
                                          passwordVariable: 'DOCKERHUB_PASSWORD')]) {
          sh 'echo "$DOCKERHUB_PASSWORD" | docker login -u "$DOCKERHUB_USER" --password-stdin'
        }
      }
    }

    stage('Build & Push Backend') {
      steps {
        dir('backend') {
          script {
            sh """
              set -e
              echo "Building ${IMAGE_BACKEND}:${BUILD_NUMBER}"
              docker build -t ${IMAGE_BACKEND}:${BUILD_NUMBER} .
              docker push ${IMAGE_BACKEND}:${BUILD_NUMBER}
              docker tag ${IMAGE_BACKEND}:${BUILD_NUMBER} ${IMAGE_BACKEND}:latest
              docker push ${IMAGE_BACKEND}:latest
            """
          }
        }
      }
    }

    stage('Build & Push Frontend') {
      steps {
        dir('frontend') {
          script {
            sh """
              set -e
              echo "Building ${IMAGE_FRONTEND}:${BUILD_NUMBER}"
              docker build -t ${IMAGE_FRONTEND}:${BUILD_NUMBER} .
              docker push ${IMAGE_FRONTEND}:${BUILD_NUMBER}
              docker tag ${IMAGE_FRONTEND}:${BUILD_NUMBER} ${IMAGE_FRONTEND}:latest
              docker push ${IMAGE_FRONTEND}:latest
            """
          }
        }
      }
    }
      stage('Deploy with Docker Compose') {
          steps { 
             script { 
             sh """ 
             set -e 
             echo "Deploying via host docker compose..." 
             cd ${COMPOSE_PATH} 
             docker compose pull 
             docker compose up -d --remove-orphans
           """
         } 
       } 
     } 
   }
     
           

  post {
    always {
      
      sh 'docker logout || true'
    }
    success {
      echo "Pipeline completed successfully."
    }
    failure {
      echo "Pipeline failed. Check console output for details."
    }
  }
}

