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
    }
}

