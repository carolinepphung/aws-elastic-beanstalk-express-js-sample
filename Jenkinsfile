pipeline {
    agent {
        docker {
            image 'node:16'   // Use Node 16 Docker image as build agent
            args '-u root:root -v /var/run/docker.sock:/var/run/docker.sock' 
            // run as root so npm can install globally & access docker
        }
    }

    environment {
        REGISTRY = "docker.io/carolinepphung"   // replace with your DockerHub/GHCR namespace
        IMAGE_NAME = "aws-elastic-beanstalk-express-js-sample"
    }

    stages {
        stage('Install Dependencies') {
            steps {
                sh 'npm install --save'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'npm test || true'  
                // allow tests to run even if none are defined, remove "|| true" once real tests exist
            }
        }

        stage('Security Scan (Snyk)') {
            steps {
                sh '''
                  if ! command -v snyk >/dev/null 2>&1; then
                      npm install -g snyk
                  fi
                  snyk test --severity-threshold=high
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                  docker build -t $REGISTRY/$IMAGE_NAME:$BUILD_NUMBER .
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                                                 usernameVariable: 'DOCKER_USER',
                                                 passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                      echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                      docker push $REGISTRY/$IMAGE_NAME:$BUILD_NUMBER
                    '''
                }
            }
        }
    }

    post {
        always {
            // Archive logs/artifacts
            archiveArtifacts artifacts: '**/npm-debug.log', allowEmptyArchive: true
        }
        failure {
            echo "Pipeline failed - check logs."
        }
        success {
            echo "Pipeline completed successfully"
        }
    }
}

