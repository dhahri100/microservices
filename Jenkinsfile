pipeline {
    agent any
    parameters {
        string(name: 'APP_NAME', defaultValue: '', description: 'Name of the service to deploy')
        string(name: 'IMAGE_TAG', defaultValue: '', description: 'Docker image tag for the service')
    }
    stages {
        stage('Prepare Environment') {
            steps {
                script {
                    if (params.APP_NAME && params.IMAGE_TAG) {
                        sh """
                            # Ensure namespace exists
                            kubectl get namespace microservices || kubectl create namespace microservices
                            
                            # Update values.yaml with new image tag for the specific service
                            yq e '.${params.APP_NAME}.image.tag = "${params.IMAGE_TAG}"' -i ./helm-chart/values.yaml
                        """
                    } else {
                        error "APP_NAME and IMAGE_TAG parameters are required"
                    }
                }
            }
        }
        stage('Deploy Service') {
            steps {
                script {
                    if (params.APP_NAME && params.IMAGE_TAG) {
                        withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: 'gke', contextName: '', credentialsId: '', namespace: 'microservices', serverUrl: 'http://34.46.5.46']]) {
                            sh """
                                helm upgrade --install microservices-release ./helm-chart -f ./helm-chart/values.yaml \
                                --namespace microservices
                            """
                        }
                    } else {
                        error "APP_NAME and IMAGE_TAG parameters are required"
                    }
                }
            }
        }
    }
}
