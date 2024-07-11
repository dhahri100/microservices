pipeline {
    agent any

    environment {
        
        CHART_REPO = 
        CHART_NAME = 
        NAMESPACE = 
        GKE_CLUSTER = 
        GKE_ZONE = 
        PROJECT_ID = 
        SLACK_CHANNEL = 
        SLACK_CREDENTIALS_ID = 
        DOCKER_REGISTRY = "mouhib543"
         = 
         = 
    }

    stages {
        stage('Setup GKE') {
            steps {
                script {
                    // Authenticate to GCP and set up GKE
                    sh """
                    gcloud auth activate-service-account --key-file=/path/to/your/keyfile.json
                    gcloud config set project ${PROJECT_ID}
                    gcloud container clusters get-credentials ${GKE_CLUSTER} --zone ${GKE_ZONE}
                    """
                }
            }
        }

        stage('Deploy with Helm') {
            steps {
                script {
                    // Deploy the application using Helm
                    sh """
                        helm upgrade --install emailservice ${}/emailservice --namespace default --set image.repository=${DOCKER_REGISTRY}/emailservice --set image.tag=latest
                        helm upgrade --install checkoutservice ${}/checkoutservice --namespace default --set image.repository=${DOCKER_REGISTRY}/checkoutservice --set image.tag=latest
                        helm upgrade --install recommendationservice ${}/recommendationservice --namespace default --set image.repository=${DOCKER_REGISTRY}/recommendationservice --set image.tag=latest
                        helm upgrade --install frontend ${}/frontend --namespace default --set image.repository=${DOCKER_REGISTRY}/frontend --set image.tag=latest
                        helm upgrade --install paymentservice ${}/paymentservice --namespace default --set image.repository=${DOCKER_REGISTRY}/paymentservice --set image.tag=latest
                        helm upgrade --install productcatalogservice ${}/productcatalogservice --namespace default --set image.repository=${DOCKER_REGISTRY}/productcatalogservice --set image.tag=latest
                        helm upgrade --install cartservice ${}/cartservice --namespace default --set image.repository=${DOCKER_REGISTRY}/cartservice --set image.tag=latest
                        helm upgrade --install loadgenerator ${}/loadgenerator --namespace default --set image.repository=${DOCKER_REGISTRY}/loadgenerator --set image.tag=latest
                        helm upgrade --install currencyservice ${}/currencyservice --namespace default --set image.repository=${DOCKER_REGISTRY}/currencyservice --set image.tag=latest
                        helm upgrade --install shippingservice ${}/shippingservice --namespace default --set image.repository=${DOCKER_REGISTRY}/shippingservice --set image.tag=latest
                        helm upgrade --install adservice ${}/adservice --namespace default --set image.repository=${DOCKER_REGISTRY}/adservice --set image.tag=latest
                    """
                }
            }
        }

        stage('ArgoCD Canary Release') {
            steps {
                script {
                    // Sync Argo CD application for canary release
                    sh """
                        argocd app sync emailservice --namespace ${}
                        argocd app sync checkoutservice --namespace ${}
                        argocd app sync recommendationservice --namespace ${}
                        argocd app sync frontend --namespace ${}
                        argocd app sync paymentservice --namespace ${}
                        argocd app sync productcatalogservice --namespace ${}
                        argocd app sync cartservice --namespace ${}
                        argocd app sync loadgenerator --namespace ${}
                        argocd app sync currencyservice --namespace ${}
                        argocd app sync shippingservice --namespace ${}
                        argocd app sync adservice --namespace ${}
                    """
                }
            }
        }

    }    
    

    post {
        always {
            script {
                // Notify Slack channel
                slackSend (
                    channel: SLACK_CHANNEL,
                    color: currentBuild.result == 'SUCCESS' ? 'good' : 'danger',
                    message: "Build ${currentBuild.fullDisplayName} finished with status ${currentBuild.result}"
                )
            }
        }
    
        failure {
            script {
                // Notify Slack on failure
                slackSend (
                    channel: SLACK_CHANNEL,
                    color: 'danger',
                    message: "Build ${currentBuild.fullDisplayName} failed."
                )
            }
        }

        success {
            script {
                // Notify Slack on success
                slackSend (
                    channel: SLACK_CHANNEL,
                    color: 'good',
                    message: "Build ${currentBuild.fullDisplayName} succeeded."
                )
            }
        }
    }

}
