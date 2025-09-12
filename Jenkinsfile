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
  }
  
  post {
    success {
      archiveArtifacts artifacts: 'build/**'
    }
  }
}