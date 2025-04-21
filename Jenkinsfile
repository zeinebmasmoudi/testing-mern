pipeline {
    agent any
    
    environment {
        TAG = "${env.BUILD_NUMBER}"
        DOCKER_COMPOSE_FILE = "docker-compose.yml"
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build Backend Image') {
            steps {
                dir('server') {
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        bat "docker build -t %DOCKER_USERNAME%/express-backend:%TAG% ."
                    }
                }
            }
        }
        
        stage('Build Frontend Image') {
            steps {
                dir('client') {
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        bat "docker build -t %DOCKER_USERNAME%/react-frontend:%TAG% ."
                    }
                }
            }
        }
        
        stage('Push Images to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    bat "echo %DOCKER_PASSWORD%| docker login -u %DOCKER_USERNAME% --password-stdin"
                    bat "docker push %DOCKER_USERNAME%/express-backend:%TAG%"
                    bat "docker push %DOCKER_USERNAME%/react-frontend:%TAG%"
                    
                    // Also tag and push as latest
                    bat "docker tag %DOCKER_USERNAME%/express-backend:%TAG% %DOCKER_USERNAME%/express-backend:latest"
                    bat "docker tag %DOCKER_USERNAME%/react-frontend:%TAG% %DOCKER_USERNAME%/react-frontend:latest"
                    bat "docker push %DOCKER_USERNAME%/express-backend:latest"
                    bat "docker push %DOCKER_USERNAME%/react-frontend:latest"
                }
            }
        }
        
        stage('Deploy') {
            steps {
                // Export environment variables for docker-compose
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    bat "set DOCKER_USERNAME=%DOCKER_USERNAME%"
                    bat "set TAG=%TAG%"
                    bat "docker-compose -f %DOCKER_COMPOSE_FILE% down || echo 'No containers to stop'"
                    bat "docker-compose -f %DOCKER_COMPOSE_FILE% up -d"
                }
            }
        }
    }
    
    post {
        always {
            bat "docker logout"
            cleanWs()
        }
    }
}