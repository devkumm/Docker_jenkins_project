pipeline {
  agent any

  stages {
    stage('Build') {
      steps {
        sh 'docker build -t my-flask-app .'
        sh 'docker tag my-flask-app $DOCKER_BFLASK_IMAGE'
      }
    }
  }
}
