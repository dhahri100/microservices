pipeline {
    agent any

    environment {
        SCANNER_HOME = tool "SonarQube-Scanner"  // Assuming 'SonarQube-Scanner' tool is configured in Jenkins
        APP_NAME = "microservices-frontend"  // Name of the frontend service
        RELEASE = "1.0.0"
        DOCKER_USER = "mouhib543"  // Your DockerHub username
        DOCKER_PASS = "dockerhub"  
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
    }

    stages {

        /*stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv('SonarQube-Server') {
                        sh "${SCANNER_HOME}/bin/sonar-scanner"  // Execute SonarQube scanner
                    }
                }
            }
        }

        stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true  // Wait for SonarQube Quality Gate result
                }
            }
        }*/

        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image using the Dockerfile in the directory
                    docker.build("${DOCKER_USER}/${APP_NAME}:${IMAGE_TAG}")
                }
            }
        }

        stage("Trivy Image Scan") {
            steps {
                script {
                    // Perform Trivy image scan
                    sh "docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image ${DOCKER_USER}/${APP_NAME}:${IMAGE_TAG} --no-progress --exit-code 0 --severity HIGH,CRITICAL --format table --scanners vuln --timeout 50m > trivy_${APP_NAME}.txt"
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('', DOCKER_PASS) {
                        // Push the Docker image to Docker Hub
                        docker.image("${DOCKER_USER}/${APP_NAME}:${IMAGE_TAG}").push()
                    }
                }
            }
        }

        stage ('Cleanup Artifact') {
            steps {
                script {
                    sh "docker rmi ${DOCKER_USER}/${APP_NAME}:${IMAGE_TAG}"  // Remove Docker image
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: "trivy_${APP_NAME}.txt", allowEmptyArchive: true  // Archive Trivy scan results
        }
    }
}
