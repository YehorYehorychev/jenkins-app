pipeline {
  agent any

  environment {
    NETLIFY_SITE_ID = 'c9bf685c-a128-49ab-bb82-701e368414a4'
    NETLIFY_AUTH_TOKEN = credentials('netlify_token')
    REACT_APP_VERSION = "1.0.$BUILD_ID"
  }

  stages {
    
    stage('Build') {
      agent {
        docker {
          image 'node:18-alpine'
          reuseNode true
          args '-u root'
        }
      }
      steps {
        sh '''
          ls -la
          node --version
          npm --version
          npm ci
          npm run build
        '''
      }
    }

    stage('Unit Tests') {
      agent {
        docker {
          image 'node:18-alpine'
          reuseNode true
          args '-u root'
        }
      }
      steps {
        sh '''
          test -f build/index.html
          npm test -- --ci --reporters=default --reporters=jest-junit
        '''
      }
      post {
        always {
          junit 'jest-results/junit.xml'
        }
      }
    }

    stage('E2E Tests - Local') {
      agent {
        docker {
          image 'playwright-jammy'
          reuseNode true
          args '-u root'
        }
      }
      steps {
        sh '''
          serve -s build -l 3000 &
          SERVER_PID=$!
          echo "Server PID=$SERVER_PID"
          sleep 5

          npx playwright test --reporter=html || true

          kill $SERVER_PID
        '''
      }
      post {
        always {
          publishHTML([
            allowMissing: false,
            alwaysLinkToLastBuild: true,
            keepAll: true,
            reportDir: 'playwright-report',
            reportFiles: 'index.html',
            reportName: 'Playwright - Local Report',
            useWrapperFileDirectly: false
          ])
          archiveArtifacts artifacts: 'playwright-report/**'
        }
      }
    }

    stage('Deploy & E2E Tests - Staging') {
      agent {
        docker {
          image 'playwright-jammy'
          reuseNode true
          args '-u root'
        }
      }
      steps {
        sh '''
          netlify --version
          
          echo "Deploying to Staging. Site ID: $NETLIFY_SITE_ID"
          netlify status
          netlify deploy --dir=build --json > deploy-output.json
          node-jq -r '.deploy_url' deploy-output.json > staging_url.txt

          STAGING_URL=$(cat staging_url.txt)
          echo "Staging deployed at: $STAGING_URL"

          echo "Running E2E tests against staging: $STAGING_URL"
          CI_ENVIRONMENT_URL=$STAGING_URL npx playwright test --project=staging --reporter=html || true
        '''
      }
      post {
        always {
          publishHTML([
            allowMissing: false,
            alwaysLinkToLastBuild: true,
            keepAll: true,
            reportDir: 'playwright-report',
            reportFiles: 'index.html',
            reportName: 'Playwright - Staging Report',
            useWrapperFileDirectly: false
          ])
          archiveArtifacts artifacts: 'playwright-report/**'
        }
      }
    }

    stage('Deploy & E2E Tests - Prod') {
      agent {
        docker {
          image 'playwright-jammy'
          reuseNode true
          args '-u root'
        }
      }
      environment {
        CI_ENVIRONMENT_URL = 'https://voluble-stroopwafel-6730a7.netlify.app'
      }
      steps {
        sh '''
          node --version
          netlify --version
          
          echo "Deploying to Production. Site ID: $NETLIFY_SITE_ID"
          netlify status
          netlify deploy --dir=build --prod
          
          echo "Running E2E tests against production: $CI_ENVIRONMENT_URL"
          npx playwright test --reporter=html || true
        '''
      }
      post {
        always {
          publishHTML([
            allowMissing: false,
            alwaysLinkToLastBuild: true,
            keepAll: true,
            reportDir: 'playwright-report',
            reportFiles: 'index.html',
            reportName: 'Playwright - Prod Report',
            useWrapperFileDirectly: false
          ])
          archiveArtifacts artifacts: 'playwright-report/**'
        }
      }
    }
  }

  post {
    success {
      archiveArtifacts artifacts: 'build/**'
    }
  }
}