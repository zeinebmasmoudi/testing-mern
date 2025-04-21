pipeline {
  agent {
    docker {
      image 'docker:24.0'
      args '--privileged -v /var/run/docker.sock:/var/run/docker.sock'
    }
  }

  environment {
    // Use your exact DockerHub username from credentials
    DOCKER_REGISTRY = 'zeineb092'
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build & Push') {
      steps {
        script {
          withCredentials([usernamePassword(
            credentialsId: 'docker-hub-credentials',
            usernameVariable: 'DOCKERHUB_USER',
            passwordVariable: 'DOCKERHUB_PASS'
          )]) {
            // Backend
            dir('server') {
              retry(3) {
                sh """
                  docker build -t ${DOCKER_REGISTRY}/mern-backend:${GIT_COMMIT_SHORT} .
                  echo \"Logging into DockerHub as ${DOCKERHUB_USER}\"
                  echo \"${DOCKERHUB_PASS}\" | docker login -u \"${DOCKERHUB_USER}\" --password-stdin
                  docker push ${DOCKER_REGISTRY}/mern-backend:${GIT_COMMIT_SHORT}
                """
              }
            }

            // Frontend
            dir('client') {
              retry(3) {
                sh """
                  docker build -t ${DOCKER_REGISTRY}/mern-frontend:${GIT_COMMIT_SHORT} .
                  docker push ${DOCKER_REGISTRY}/mern-frontend:${GIT_COMMIT_SHORT}
                """
              }
            }
          }
        }
      }
    }

    stage('Deploy') {
      steps {
        sh '''
          docker-compose down || true
          docker-compose pull
          docker-compose up -d
        '''
      }
    }
  }

  post {
    always {
      sh 'docker logout || true'
      cleanWs()
    }
    failure {
      slackSend(
        channel: '#build-alerts',
        message: "FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
      )
    }
  }
}