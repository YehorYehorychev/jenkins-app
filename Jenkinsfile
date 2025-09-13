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
          echo ">>> Files in workspace"
          ls -la

          echo ">>> Node & NPM versions"
          node --version
          npm --version

          echo ">>> Installing dependencies"
          npm ci

          echo ">>> Running build"
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
          image 'mcr.microsoft.com/playwright:v1.55.0-noble'
          reuseNode true
          args '-u root'
        }
      }
      steps {
        sh '''
          npm install --save-dev serve
          node_modules/.bin/serve -s build &
          npx playwright test
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