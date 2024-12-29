 def manifestFile = 'kubernetes-manifest.yaml'
 def namespace = 'default'
pipeline {
  agent any

  stages {
    stage('Build') {
      steps {
        sh 'docker build -t my-flask-app .'
        sh 'docker tag my-flask-app $DOCKER_BFLASK_IMAGE'
      }
    }
    stage('Deploy') {
      steps {
        withCredentials([usernamePassword(credentialsId: "${DOCKER_REGISTRY_CREDS}", passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
          sh "echo \$DOCKER_PASSWORD | docker login -u \$DOCKER_USERNAME --password-stdin docker.io"
          sh 'docker push $DOCKER_BFLASK_IMAGE'
        }
      }
    }
   stage('Run Container') {  
      steps {
        sh 'docker run -dit --name test_image $DOCKER_BFLASK_IMAGE'
      }
    }
   stage('Verify Container') {
      steps {
        sh 'docker exec -i test_image uptime'
      }
    }
   stage('Cleanup') {
      steps {
        sh 'docker stop test_image'
        sh 'docker rm test_image'
      }
    }
   stage('Apply Kubernetes manifest') {
      steps {
        withCredentials([file(credentialsId: '43f13e92-4bbb-4c64-b13f-4502bdf7f74b', variable: 'Kube_config')]) {
          script {
            sh """
            sed \
                -e "s|{{NAMESPACE}}|${namespace}|g" \
                -e "s|{{PULL_IMAGE}}|${DOCKER_BFLASK_IMAGE}|g" \
                ${manifestFile} \
            | kubectl --kubeconfig=${Kube_config} apply -f -
            """
          }
        }
      }
    }
  }
}
