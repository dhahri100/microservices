pipeline {
    agent any

    environment {
        APP_NAME = "microservices-application"
        CHART_REPO = "your-helm-chart-repo"
        CHART_NAME = "your-helm-chart-name"
        NAMESPACE = "your-namespace"
        GKE_CLUSTER = "your-gke-cluster"
        GKE_ZONE = "your-gke-zone"
        PROJECT_ID = "your-gcp-project-id"
        SLACK_CHANNEL = '#your-slack-channel'
        SLACK_CREDENTIALS_ID = 'your-slack-credentials-id'
        DOCKER_REGISTRY = "your-docker-registry"
        HELM_CHART_DIR = "your-helm-chart-directory"
        ARGOCD_NAMESPACE = "your-argocd-namespace"
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

        stages {
        stage('Deploy To Kubernetes') {
            steps {
                
                    sh "kubectl apply -f deployment-service.yml"
                    
                }
            }
        }
        
        stage('verify Deployment') {
            steps {
                
                    sh "kubectl get svc -n webapps"
                }
            }
        }
    }

        stage('Deploy with Helm') {
            steps {
                script {
                    // Deploy the application using Helm
                    sh """
                        helm upgrade --install emailservice ${HELM_CHART_DIR}/emailservice --namespace default --set image.repository=${DOCKER_REGISTRY}/emailservice --set image.tag=latest
                        helm upgrade --install checkoutservice ${HELM_CHART_DIR}/checkoutservice --namespace default --set image.repository=${DOCKER_REGISTRY}/checkoutservice --set image.tag=latest
                        helm upgrade --install recommendationservice ${HELM_CHART_DIR}/recommendationservice --namespace default --set image.repository=${DOCKER_REGISTRY}/recommendationservice --set image.tag=latest
                        helm upgrade --install frontend ${HELM_CHART_DIR}/frontend --namespace default --set image.repository=${DOCKER_REGISTRY}/frontend --set image.tag=latest
                        helm upgrade --install paymentservice ${HELM_CHART_DIR}/paymentservice --namespace default --set image.repository=${DOCKER_REGISTRY}/paymentservice --set image.tag=latest
                        helm upgrade --install productcatalogservice ${HELM_CHART_DIR}/productcatalogservice --namespace default --set image.repository=${DOCKER_REGISTRY}/productcatalogservice --set image.tag=latest
                        helm upgrade --install cartservice ${HELM_CHART_DIR}/cartservice --namespace default --set image.repository=${DOCKER_REGISTRY}/cartservice --set image.tag=latest
                        helm upgrade --install loadgenerator ${HELM_CHART_DIR}/loadgenerator --namespace default --set image.repository=${DOCKER_REGISTRY}/loadgenerator --set image.tag=latest
                        helm upgrade --install currencyservice ${HELM_CHART_DIR}/currencyservice --namespace default --set image.repository=${DOCKER_REGISTRY}/currencyservice --set image.tag=latest
                        helm upgrade --install shippingservice ${HELM_CHART_DIR}/shippingservice --namespace default --set image.repository=${DOCKER_REGISTRY}/shippingservice --set image.tag=latest
                        helm upgrade --install adservice ${HELM_CHART_DIR}/adservice --namespace default --set image.repository=${DOCKER_REGISTRY}/adservice --set image.tag=latest
                    """
                }
            }
        }

        stage('ArgoCD Canary Release') {
            steps {
                script {
                    // Sync Argo CD application for canary release
                    sh """
                        argocd app sync emailservice --namespace ${ARGOCD_NAMESPACE}
                        argocd app sync checkoutservice --namespace ${ARGOCD_NAMESPACE}
                        argocd app sync recommendationservice --namespace ${ARGOCD_NAMESPACE}
                        argocd app sync frontend --namespace ${ARGOCD_NAMESPACE}
                        argocd app sync paymentservice --namespace ${ARGOCD_NAMESPACE}
                        argocd app sync productcatalogservice --namespace ${ARGOCD_NAMESPACE}
                        argocd app sync cartservice --namespace ${ARGOCD_NAMESPACE}
                        argocd app sync loadgenerator --namespace ${ARGOCD_NAMESPACE}
                        argocd app sync currencyservice --namespace ${ARGOCD_NAMESPACE}
                        argocd app sync shippingservice --namespace ${ARGOCD_NAMESPACE}
                        argocd app sync adservice --namespace ${ARGOCD_NAMESPACE}
                    """
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


