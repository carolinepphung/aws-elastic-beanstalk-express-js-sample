pipeline {
    agent {
        docker {
            image 'node:16'
            args '-u root:root' // run as root so npm install works
        }
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/carolinepphung/aws-elastic-beanstalk-express-js-sample.git'
            }
        }
        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }
        stage('Snyk Security Scan') {
            steps {
                sh 'npx snyk test || true'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t my-app:latest .'
            }
        }
        stage('Push Docker Image') {
            environment {
                DOCKER_HUB_CREDENTIALS = credentials('dockerhub-creds')
            }
            steps {
                sh '''
                  echo "$DOCKER_HUB_CREDENTIALS_PSW" | docker login -u "$DOCKER_HUB_CREDENTIALS_USR" --password-stdin
                  docker tag my-app:latest mydockerhubuser/my-app:latest
                  docker push mydockerhubuser/my-app:latest
                '''
            }
        }
    }
    post {
        always {
            echo 'Cleaning up local docker state...'
            sh 'docker system prune -af || true'
        }
        failure {
            echo 'Pipeline failed — check console output.'
        }
    }
}

