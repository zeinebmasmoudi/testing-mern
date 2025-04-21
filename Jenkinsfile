pipeline {
  agent any

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build & Push Containers') {
      environment {
        // Correct credential ID with hyphens
        DOCKER_CREDS = credentials('docker-hub-credentials')
      }
      steps {
        script {
          // Backend
          dir('server') {
            withCredentials([usernamePassword(
              credentialsId: 'docker-hub-credentials',
              usernameVariable: 'DOCKER_USER',
              passwordVariable: 'DOCKER_PASS'
            )]) {
              sh """
                docker build -t yourdockerhub/backend:${env.GIT_COMMIT_SHORT} .
                echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                docker push yourdockerhub/backend:${env.GIT_COMMIT_SHORT}
              """
            }
          }

          // Frontend
          dir('client') {
            withCredentials([usernamePassword(
              credentialsId: 'docker-hub-credentials',
              usernameVariable: 'DOCKER_USER',
              passwordVariable: 'DOCKER_PASS'
            )]) {
              sh """
                docker build -t yourdockerhub/frontend:${env.GIT_COMMIT_SHORT} .
                echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                docker push yourdockerhub/frontend:${env.GIT_COMMIT_SHORT}
              """
            }
          }
        }
      }
    }

    stage('Deploy') {
      steps {
        sh '''
          docker-compose pull
          docker-compose down --remove-orphans
          docker-compose up -d
        '''
      }
    }
  }

  post {
    always {
      sh 'docker logout'
    }
  }
}