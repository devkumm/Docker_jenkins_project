pipeline {
    agent any
    
    environment {
        K8S_NAMESPACE = 'default'   // Kubernetes namespace
        MANIFEST_FILE = 'kubernetes-manifest.yaml'   // Kubernetes manifest file
        // DOCKER_BFLASK_IMAGE = 'your_docker_image_name'   // Add your Docker image name if missing
    }

    stages {
        stage('Build') {
            steps {
                script {
                    // Build Docker image
                    sh 'docker build -t my-flask-app .'
                    
                    // Tag the image
                    sh 'docker tag my-flask-app $DOCKER_BFLASK_IMAGE'
                }
            }
        }

        stage('Deploy') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKER_REGISTRY_CREDS}", passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    script {
                        // Login to Docker registry and push the image
                        sh "echo \$DOCKER_PASSWORD | docker login -u \$DOCKER_USERNAME --password-stdin docker.io"
                        sh 'docker push $DOCKER_BFLASK_IMAGE'
                    }
                }
            }
        }

        stage('Run Container') {
            steps {
                script {
                    // Run the Docker container
                    sh 'docker run -dit --name test_image $DOCKER_BFLASK_IMAGE'
                }
            }
        }

        stage('Verify Container') {
            steps {
                script {
                    // Verify the container is running
                    sh 'docker exec -i test_image uptime'
                }
            }
        }

        stage('Cleanup') {
            steps {
                script {
                    // Stop and remove the test container
                    sh 'docker stop test_image'
                    sh 'docker rm test_image'
                }
            }
        }

        stage('Apply Kubernetes manifest') {
            steps {
                withCredentials([file(credentialsId: '43f13e92-4bbb-4c64-b13f-4502bdf7f74b', variable: 'Kube_config')]) {
                    script {
                        // Substitute values in the manifest and apply to Kubernetes
sh """
    result=\$(sed -e "s|{{NAMESPACE}}|${K8S_NAMESPACE}|g" -e "s|{{PULL_IMAGE}}|${DOCKER_BFLASK_IMAGE}|g" ${MANIFEST_FILE})
    echo "\$result"
    echo "\$result" | kubectl apply -f - --validate=false
"""

                    }
                }
            }
        }
    }
}

