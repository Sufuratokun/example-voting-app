pipeline {
  agent any
  stages {
    stage('Checkout Code') {
      steps {
        checkout scm
      }
    }
    stage('Setup Tools') {
      steps {
        sh 'curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"'
        sh 'chmod +x ./kubectl'
      }
    }
    stage('Deploy to k3d') {
      steps {
        sh "./kubectl apply -f k8s-specifications/'
      }
    }
  }
}
