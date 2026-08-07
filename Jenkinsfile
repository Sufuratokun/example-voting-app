pipeline {
  agent any
  stages {
    stage('Checkout Code') {
      steps {
        checkout scm
      }
    }
    stage('Test Connection') {
      steps {
        echo 'Успех! Jenkins получил доступ к репозиторю.'
        sh 'ls -la'
      }
    }
  }
}
