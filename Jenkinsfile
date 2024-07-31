pipeline {
    agent any
    parameters {
        string(name: 'APP_NAME', defaultValue: '', description: 'Name of the service to deploy')
        string(name: 'IMAGE_TAG', defaultValue: '', description: 'Docker image tag for the service')
    }
    stages {
        stage('Prepare Deployment') {
            steps {
                script {
                    if (params.APP_NAME && params.IMAGE_TAG) {
                        withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: 'gke', contextName: '', credentialsId: '', namespace: 'microservices', serverUrl: 'http://34.46.5.46']]) {
                        // Perform Helm upgrade or install
                        sh """
                            helm upgrade --install ${params.APP_NAME} ./helm/${params.APP_NAME} \
                            --set image.repository=mouhib543/${params.APP_NAME} \
                            --set image.tag=${params.IMAGE_TAG}
                        """
                    } 
                    else {
                        error "SERVICE_NAME and IMAGE_TAG parameters are required"
                    }
                }
            }
        }
        
    }
}
}