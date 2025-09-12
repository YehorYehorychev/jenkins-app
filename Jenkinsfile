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
          echo ">>> Checking files in workspace"
          ls -la

          echo ">>> Node & NPM versions"
          node --version
          npm --version

          if [ -f package-lock.json ]; then
            echo ">>> Found package-lock.json, running npm ci"
            npm ci
          else
            echo ">>> No package-lock.json, falling back to npm install"
            npm install
          fi

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