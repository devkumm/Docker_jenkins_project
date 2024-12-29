pipeline {
  agent any
environment {
        K8S_NAMESPACE = 'default' // Kubernetes namespace
        MANIFEST_FILE = 'kubernetes-manifest.yaml' // Kubernetes manifest file
       // DOCKER_IMAGE = 'env.DOCKER_BFLASK_IMAGE'  // Fallback Docker image
    }
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
   stage('Cleanup1') {
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
                            -e "s|{{NAMESPACE}}|${K8S_NAMESPACE}|g" \
                            -e "s|{{PULL_IMAGE}}|${DOCKER_BFLASK_IMAGE}|g" \
                            ${MANIFEST_FILE} \
                        | kubectl apply -f -
                        """
          }
        }
      }
    }
  }
}
