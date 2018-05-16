pipeline {
  agent {
    node {
      label 'build'
    }

  }
  stages {
    stage('Build') {
      steps {
        git 'https://github.com/pmaugeri/ak-devops-lab'
      }
    }
  }
}