pipeline {
  agent any

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
        sh '''
        set -euo pipefail
        echo "=== DEPLOY STAGE START ==="
        echo "whoami: $(whoami)"
        echo "WORKSPACE=${WORKSPACE}"
        echo "PATH=$PATH"
        echo "pwd: $(pwd)"
        echo "Listing WORKSPACE contents:"
        ls -la "${WORKSPACE}" || true

        echo "Checking docker and compose availability..."
        if command -v docker >/dev/null 2>&1; then
          echo "docker: $(docker --version)"
        else
          echo "ERROR: docker not found in this agent. Exiting."
          exit 127
        fi

        if command -v docker-compose >/dev/null 2>&1; then
          DC_BIN=$(command -v docker-compose)
          echo "Found docker-compose at: $DC_BIN"
          $DC_BIN --version || true
          COMPOSE_CMD="$DC_BIN"
        elif docker compose version >/dev/null 2>&1; then
          echo "Found docker compose plugin (docker compose)"
          COMPOSE_CMD="docker compose"
        else
          echo "docker-compose not found; will use docker/compose container fallback"
          COMPOSE_CMD="docker run --rm -v /var/run/docker.sock:/var/run/docker.sock -v \"${WORKSPACE}\":/workdir -w /workdir docker/compose:2.20.2"
        fi

        echo "Using compose command: $COMPOSE_CMD"

        # go to your compose folder inside workspace (use WORKSPACE rather than hardcoded path)
        cd "${WORKSPACE}" || exit 1
        # if your compose file is in a subfolder, change dir('path') or cd accordingly

        echo "Running: $COMPOSE_CMD pull"
        # shell-split to allow both forms (binary or `docker compose`) and the fallback container
        sh -c "$COMPOSE_CMD pull"

        echo "Running: $COMPOSE_CMD up -d --remove-orphans"
        sh -c "$COMPOSE_CMD up -d --remove-orphans"

        echo "=== DEPLOY STAGE END ==="
      '''
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

