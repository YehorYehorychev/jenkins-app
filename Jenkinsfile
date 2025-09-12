pipeline {
  agent any
  options {
      timeout(time: 10, unit: 'SECONDS')
  }

  stages {
    stage('Workspace Cleanup') {
      steps {
        echo 'Cleaning the workspace..'
        cleanWs()
      }
    }

    stage('Install Node:18 and Build the App') {
      agent {
        docker {
          image 'node:18-alpine'
          reuseNode true
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
  }
  
  post {
    success {
      archiveArtifacts artifacts: 'build/**'
    }
  }
}