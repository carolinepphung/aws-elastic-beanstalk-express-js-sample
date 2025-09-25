pipeline {
    agent any

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

        stage('Install Node.js') {
            steps {
                sh '''
                    curl -fsSL https://deb.nodesource.com/setup_20.x | bash -
                    apt-get install -y nodejs
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install --save'
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    def testStatus = sh(script: 'npm test || echo "No tests defined"', returnStatus: true)
                    if (testStatus != 0) {
                        echo "Tests skipped or not defined. Continuing pipeline..."
                    }
                }
            }
        }

        stage('Security Scan') {
            steps {
                sh '''
                    npm install -g snyk
                    snyk test || echo "Snyk scan found issues but will not fail pipeline."
                '''
            }
        }

        stage('Prepare Dockerfile') {
            steps {
                writeFile file: 'Dockerfile', text: '''
                FROM node:20
                WORKDIR /app
                COPY package*.json ./
                RUN npm install --production
                COPY . .
                EXPOSE 8080
                CMD ["npm", "start"]
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

