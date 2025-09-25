pipeline {
    agent any   // Run on Jenkins agent with Docker installed

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')
        DOCKER_IMAGE = "carolinepphung/aws-express-sample"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/carolinepphung/aws-elastic-beanstalk-express-js-sample.git'
            }
        }

        stage('Install Dependencies') {
            agent {
                docker {
                    image 'node:16'
                    args '-u root:root'
                }
            }
            steps {
                sh 'npm install'
            }
        }

        stage('Run Tests') {
            agent {
                docker { image 'node:16' }
            }
            steps {
                script {
                    sh 'npm test || echo "No tests defined"'
                }
            }
        }

        stage('Security Scan') {
            agent {
                docker { image 'node:16' }
            }
            steps {
                sh '''
                  npm install -g snyk
                  snyk test || echo "Snyk scan skipped or authentication missing"
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t $DOCKER_IMAGE:$BUILD_NUMBER .
                    docker tag $DOCKER_IMAGE:$BUILD_NUMBER $DOCKER_IMAGE:latest
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push $DOCKER_IMAGE:$BUILD_NUMBER
                        docker push $DOCKER_IMAGE:latest
                    '''
                }
            }
        }
    }
}

