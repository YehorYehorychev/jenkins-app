pipeline {
  agent any
  
  stages {
    stage('Build with Node 18') {
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

    stage('Run Tests') {
      parallel {
        stage('Test') {
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
              npm test
            '''
          }
        }

        stage('E2E') {
          agent {
            docker {
              image 'mcr.microsoft.com/playwright:v1.55.0-jammy'
              reuseNode true
              args '-u root'
            }
          }
          steps {
            sh '''
              ./node_modules/.bin/serve -s build -l 3000 &
              SERVER_PID=$!
              echo "Server PID=$SERVER_PID"
              sleep 5
              npx playwright test --reporter=html
              kill $SERVER_PID
            '''
          }
        }
      }
    }
  }
  
  post {
    always {
      junit 'jest-results/junit.xml'
      publishHTML([
        allowMissing: false,
        alwaysLinkToLastBuild: false,
        icon: '',
        keepAll: false,
        reportDir: 'playwright-report',
        reportFiles: 'index.html',
        reportName: 'Playwright - HTML Report',
        reportTitles: '',
        useWrapperFileDirectly: true
      ])
    }
    success {
      archiveArtifacts artifacts: 'build/**'
    }
  }
}