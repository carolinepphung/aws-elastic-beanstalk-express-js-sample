pipeline {
    agent any   // uses Jenkins node directly
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/carolinepphung/aws-elastic-beanstalk-express-js-sample.git'
            }
        }
        stage('Install Dependencies') {
            steps {
                sh 'npm install --save'
            }
        }
        stage('Run Tests') {
            steps {
                sh 'npm test'
            }
        }
        stage('Security Scan') {
            steps {
                sh '''
                npm install -g snyk
                snyk test || exit 1
                '''
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t my-app:latest .'
            }
        }
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_HUB_USR', passwordVariable: 'DOCKER_HUB_PSW')]) {
                    sh '''
                    echo "$DOCKER_HUB_PSW" | docker login -u "$DOCKER_HUB_USR" --password-stdin
                    docker tag my-app:latest mydockerhubuser/my-app:latest
                    docker push mydockerhubuser/my-app:latest
                    '''
                }
            }
        }
    }
    post {
        always { echo 'Pipeline finished!' }
        success { echo 'Pipeline completed successfully!' }
        failure { echo 'Pipeline failed — check console logs.' }
    }
}

