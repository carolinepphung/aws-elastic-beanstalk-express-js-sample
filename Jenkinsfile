pipeline {
    agent any

    environment {
        DOCKER_HOST = "tcp://dind:2375"  // use DinD service from docker-compose
        IMAGE_NAME = "my-express-app"    // local image name (will be tagged before push)
    }

    stages {
        stage('Checkout') {
            steps {
                // Use your fork URL if private use credentials
                git branch: 'main', url: 'https://github.com/carolinepphung/aws-elastic-beanstalk-express-js-sample.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install --save'
            }
        }

        stage('Snyk Security Scan') {
            steps {
                // Requires a Jenkins secret text credential with id 'snyk-token'
                withCredentials([string(credentialsId: 'snyk-token', variable: 'SNYK_TOKEN')]) {
                    sh '''
                      if ! command -v snyk >/dev/null 2>&1; then
                        npm install -g snyk
                      fi
                      snyk auth "$SNYK_TOKEN"
                      snyk test --severity-threshold=high
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .'
            }
        }

        stage('Push Docker Image') {
            steps {
                // Create a Jenkins credential (username/password) with id 'dockerhub-creds'
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                                                  usernameVariable: 'DOCKERHUB_USER',
                                                  passwordVariable: 'DOCKERHUB_PASS')]) {
                    sh '''
                      echo "$DOCKERHUB_PASS" | docker login -u "$DOCKERHUB_USER" --password-stdin
                      docker tag ${IMAGE_NAME}:${BUILD_NUMBER} $DOCKERHUB_USER/${IMAGE_NAME}:${BUILD_NUMBER}
                      docker push $DOCKERHUB_USER/${IMAGE_NAME}:${BUILD_NUMBER}
                    '''
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: '**/npm-debug.log', allowEmptyArchive: true
            echo "Cleaning up local docker state (on DinD)..."
            sh 'docker system prune -af || true'
        }
        failure {
            echo "Pipeline failed — check console output."
        }
        success {
            echo "Pipeline succeeded"
        }
    }
}

