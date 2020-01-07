pipeline {
  agent {
    kubernetes {
      yamlFile 'builder.yaml'
    }
  }

  stages {
    stage('Hello from container') {
      sh 'python3 --version'
    }
  }
}
