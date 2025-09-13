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
          image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
          reuseNode true
          args '-u root'
        }
      }
      steps {
        sh '''
          ./node_modules/.bin/serve -s build &
          SERVER_PID=$!
          npx playwright test
          kill $SERVER_PID
        '''
      }
    }
  }
  
  post {
    always {
      junit 'jest-results/junit.xml'
    }
    success {
      archiveArtifacts artifacts: 'build/**'
    }
  }
}