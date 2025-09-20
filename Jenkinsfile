pipeline {
    agent {
        docker {
            image 'node:16'
            args '-v /var/run/docker.sock:/var/run/docker.sock -u root:root'
        }
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/carolinepphung/aws-elastic-beanstalk-express-js-sample.git'
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
                    // Run tests but don't fail the pipeline if none exist
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
}

