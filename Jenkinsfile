pipeline {
    agent {
        docker {
            image 'node:16'
            args '-v /var/run/docker.sock:/var/run/docker.sock'
        }
    }
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/carolinepphung/aws-elastic-beanstalk-express-js-sample.git'
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
                sh 'docker build -t carolinepphung/express-app:latest .'
            }
        }
        stage('Push Docker Image') {
            environment {
                DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')
            }
            steps {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                sh 'docker push carolinepphung/express-app:latest'
            }
        }
    }
}

