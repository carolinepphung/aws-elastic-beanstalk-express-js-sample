pipeline {
    agent {
        docker {
            image 'node:16'
            args '-u root:root -v /var/run/docker.sock:/var/run/docker.sock'
        }
    }

    environment {
        DOCKER_IMAGE = "mydockerhubuser/my-app:latest"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/carolinepphung/aws-elastic-beanstalk-express-js-sample.git'
            }
        }

        stage('Install Dependencies') {
            steps { sh 'npm install --save' }
        }

        stage('Run Tests') {
            steps { sh 'npm test' }
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
            steps { sh 'docker build -t $DOCKER_IMAGE .' }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_HUB_USR', passwordVariable: 'DOCKER_HUB_PSW')]) {
                    sh '''
                    echo "$DOCKER_HUB_PSW" | docker login -u "$DOCKER_HUB_USR" --password-stdin
                    docker push $DOCKER_IMAGE
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

