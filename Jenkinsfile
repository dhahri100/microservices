pipeline {
    agent any

    environment {
        SCANNER_HOME = tool name: 'SonarQube-Scanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
        APP_NAME = "shippingservice"
        RELEASE = "1.0.0"
        DOCKER_REPO = "microservices"
        DOCKER_PASS = "dockerhub"  
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
    }

    stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image using the Dockerfile in the service directory
                    docker.build("${DOCKER_REPO}/${APP_NAME}:${IMAGE_TAG}", "path/to/${APP_NAME}")
                }
            }
        }

        stage('Trivy Image Scan') {
            steps {
                script {
                    // Perform Trivy image scan and save results to a file
                    sh "docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image ${DOCKER_REPO}/${APP_NAME}:${IMAGE_TAG} --no-progress --exit-code 0 --severity HIGH,CRITICAL --format table --scanners vuln --timeout 50m | tee trivy_${APP_NAME}.txt"
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    // Push Docker image to Docker Hub
                    docker.withRegistry('', DOCKER_PASS) {
                        docker.image("${DOCKER_REPO}/${APP_NAME}:${IMAGE_TAG}").push()
                    }
                }
            }
        }

        stage('Cleanup Artifact') {
            steps {
                script {
                    // Remove the local Docker image
                    sh "docker rmi ${DOCKER_REPO}/${APP_NAME}:${IMAGE_TAG}"  
                }
            }
        }
    
}

    post {
        always {
            // Archive Trivy scan results
            archiveArtifacts artifacts: "trivy_${APP_NAME}.txt", allowEmptyArchive: true
        }
    }

