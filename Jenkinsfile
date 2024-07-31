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
                        // Authenticate with GKE (optional, if needed)
                        withCredentials([file(credentialsId: 'gke-credentials', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                            sh 'gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS'
                            sh 'gcloud container clusters get-credentials your-gke-cluster --zone your-gke-cluster-zone --project your-gke-project'
                        }

                        // Perform Helm upgrade or install
                        sh """
                            helm upgrade --install ${params.SERVICE_NAME} ./helm/${params.SERVICE_NAME} \
                            --set image.repository=your-dockerhub-username/${params.SERVICE_NAME} \
                            --set image.tag=${params.IMAGE_TAG}
                        """
                    } else {
                        error "SERVICE_NAME and IMAGE_TAG parameters are required"
                    }
                }
            }
        }
        
    }
}
