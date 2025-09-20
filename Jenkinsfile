pipeline {
    agent {
        docker {
            // Use Node 16 + Docker CLI installed
            image 'docker:24.0.5-dind' 
            args '-u root:root -v /var/run/docker.sock:/var/run/docker.sock'
        }
    }

    environment {
        DOCKER_HUB_CREDS = credentials('dockerhub-creds')
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', 
                    url: 'https://github.com/carolinepphung/aws-elastic-beanstalk-express-js-sample.git'
            }
        }

        stage('Install Node & Dependencies') {
            steps {
                sh '''
                # Install Node 16 in docker:dind image
                apk add --no-cache nodejs npm
                npm install --save
                '''
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
                sh '''
                echo "$DOCKER_HUB_CREDS_PSW" | docker login -u "$DOCKER_HUB_CREDS_USR" --password-stdin
                docker tag my-app:latest mydockerhubuser/my-app:latest
                docker push mydockerhubuser/my-app:latest
                '''
            }
        }
    }

    post {
        always { echo 'Pipeline finished!' }
        success { echo 'Pipeline completed successfully!' }
        failure { echo 'Pipeline failed — check console logs.' }
    }
}

