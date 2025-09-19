pipeline {
  agent {
    docker {
      image 'node:16'
      args '-u root:root' // run as root to allow npm global installs if needed
    }
  }
  environment {
    IMAGE_NAME = "YOUR_REGISTRY/YOUR_REPO/aws-express-sample:${env.BUILD_NUMBER}"
    // SNYK_TOKEN should be added as Jenkins credential and exposed as env var to job.
    SNYK_TOKEN = credentials('snyk-token') // create this credential in Jenkins (string)
    // DOCKER_REG_CRED is Jenkins credentials id for docker registry (username/password or token)
    DOCKER_REG_CRED = 'docker-registry-creds'
  }
  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Install Dependencies') {
      steps {
        sh 'npm install --save'
      }
    }

    stage('Run Unit Tests') {
      steps {
        sh '''
          if [ -f package.json ]; then
            npm test || (echo "Tests failed"; exit 1)
          else
            echo "No package.json found."
          fi
        '''
      }
    }

    stage('Security Scan - Snyk') {
      steps {
        sh '''
          # install snyk
          npm install -g snyk
          snyk auth ${SNYK_TOKEN}

          # run test; fail on high or above
          snyk test --severity-threshold=high || (echo "Snyk found high/critical issues" && exit 1)
        '''
      }
    }

    stage('Build Docker Image') {
      steps {
        // Use Docker CLI that Jenkins can access via DinD (DOCKER_HOST in env)
        sh '''
          # Build docker image
          docker build -t ${IMAGE_NAME} .
        '''
      }
    }

    stage('Push Docker Image') {
      steps {
        withCredentials([usernamePassword(credentialsId: "${DOCKER_REG_CRED}", usernameVariable: 'REG_USER', passwordVariable: 'REG_PASS')]) {
          sh '''
            echo "${REG_PASS}" | docker login -u "${REG_USER}" --password-stdin YOUR_REGISTRY
            docker push ${IMAGE_NAME}
          '''
        }
      }
    }
  }

  post {
    always {
      // archive logs and test reports if present
      archiveArtifacts artifacts: '**/target/*.log, **/npm-debug.log', allowEmptyArchive: true
      junit allowEmptyResults: true, testResults: '**/test-results/*.xml'
    }
    failure {
      mail to: 'you@example.com',
           subject: "Build ${env.JOB_NAME} #${env.BUILD_NUMBER} Failed",
           body: "Check console output at ${env.BUILD_URL}"
    }
  }
}

