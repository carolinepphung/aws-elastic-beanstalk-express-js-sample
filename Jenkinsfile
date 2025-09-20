pipeline {
    agent any  // Use the Jenkins container itself

    environment {
        DOCKER_HUB = credentials('dockerhub-creds')
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
                sh '''
                docker run --rm -v $PWD:/app -w /app node:16 npm install --save
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                docker run --rm -v $PWD:/app -w /app node:16 npm test
                '''
            }
        }

        stage('Security Scan') {
            steps {
                sh '''
                docker run --rm -v $PWD:/app -w /app node:16 npm install -g snyk
                docker run --rm -v $PWD:/app -w /app node:16 snyk test || exit 1
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
                sh '''
                echo "$DOCKER_HUB_PSW" | docker login -u "$DOCKER_HUB_USR" --password-stdin
                docker tag my-app:latest mydockerhubuser/my-app:latest
                docker push mydockerhubuser/my-app:latest
                '''
            }
        }
    }

    post {
        always { echo 'Pipeline finished!' }
        success { echo 'Pipeline succeeded!' }
        failure { echo 'Pipeline failed — check console logs.' }
    }
}

