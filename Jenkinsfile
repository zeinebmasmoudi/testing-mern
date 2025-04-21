pipeline {
  agent any

  environment {
    DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')  // Set in Jenkins credentials
    GITHUB_TOKEN = credentials('github-token')  // For private repos
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/yourusername/your-repo.git'
      }
    }

    stage('Build & Push Backend') {
      steps {
        dir('server') {
          sh 'docker build -t yourdockerhub/backend:latest .'
          sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
          sh 'docker push yourdockerhub/backend:latest'
        }
      }
    }

    stage('Build & Push Frontend') {
      steps {
        dir('client') {
          sh 'docker build -t yourdockerhub/frontend:latest .'
          sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
          sh 'docker push yourdockerhub/frontend:latest'
        }
      }
    }

    stage('Deploy') {
      steps {
        sh 'docker-compose down && docker-compose up -d'  // Deploy locally
        // For cloud (AWS ECS/Kubernetes), add deployment commands here
      }
    }
  }

  post {
    always {
      sh 'docker logout'
    }
  }
}