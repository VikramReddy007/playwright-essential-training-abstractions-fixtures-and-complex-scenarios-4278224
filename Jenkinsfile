pipeline {
  agent {
    docker {
      image 'mcr.microsoft.com/playwright:v1.41.0-jammy'
      args '-u root' // avoid permission issues
    }
  }

  parameters {
    string(name: 'TEST_TAG', defaultValue: '', description: 'Optional tag, e.g. @smoke')
  }

  options {
    timestamps()
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Install') {
      steps {
        sh 'npm ci'
        sh 'npx playwright install --with-deps'
      }
    }

    stage('Run Tests') {
      steps {
        sh """
          if [ -z "${params.TEST_TAG}" ]; then
            npx playwright test --reporter=line
          else
            npx playwright test --grep ${params.TEST_TAG} --reporter=line
          fi
        """
      }
    }
  }

  post {
    failure {
      // Upload artifacts ONLY if tests fail
      archiveArtifacts artifacts: 'playwright-report/**, test-results/**', allowEmptyArchive: true

      // (Optional but recommended)
      junit 'test-results/**/*.xml'
    }

    always {
      // Clean workspace
      cleanWs()
    }
  }
}
