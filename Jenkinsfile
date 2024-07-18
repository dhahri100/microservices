pipeline {
    agent any
    parameters {
        string(name: 'SERVICE_NAME', defaultValue: '', description: 'Name of the service to deploy')
        string(name: 'IMAGE_TAG', defaultValue: '', description: 'Docker image tag for the service')
    }
    stages {
        stage('Prepare Deployment') {
            steps {
                script {
                    if (params.SERVICE_NAME && params.IMAGE_TAG) {
                        sh "helm upgrade --install ${params.SERVICE_NAME} ./helm/${params.SERVICE_NAME} --set image.tag=${params.IMAGE_TAG}"
                    } else {
                        error "SERVICE_NAME and IMAGE_TAG parameters are required"
                    }
                }
            }
        }
        stage('Configure Istio Traffic Management') {
            steps {
                script {
                    if (params.SERVICE_NAME) {
                        // Apply Istio VirtualService and DestinationRule
                        sh "kubectl apply -f ./istio/${params.SERVICE_NAME}-virtualservice.yaml"
                        sh "kubectl apply -f ./istio/${params.SERVICE_NAME}-destinationrule.yaml"
                    }
                }
            }
        }
        stage('Deploy with Argo CD') {
            steps {
                script {
                    sh "argocd app sync ${params.SERVICE_NAME}"
                }
            }
        }
    }
}
