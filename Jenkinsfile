pipeline {
    agent any

    environment {
        SCANNER_HOME = tool name: 'SonarQube-Scanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'  // SonarQube scanner tool
        JAVA_HOME = tool name: 'JDK 11' 
        SONAR_PROJECT_KEY = "microservices-adservice"  // Specify your project key here
        SONAR_PROJECT_NAME = "Ad Service"  // Optional: Give your project a name
        SONAR_PROJECT_VERSION = "1.0"
        APP_NAME = "adservice"  // The name of the image (adservice)
        RELEASE = "1.0.0"
        DOCKER_USER = "mouhib19"  // Your DockerHub username
        DOCKER_PASS = "dockerhub"  
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
    }

    stages {
        
        /*stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv('SonarQube-Server') {
                        sh '''
                            ${SCANNER_HOME}/bin/sonar-scanner \
                            -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                            -Dsonar.projectName="${SONAR_PROJECT_NAME}" \
                            -Dsonar.projectVersion=${SONAR_PROJECT_VERSION} \
                            -Dsonar.sources=.
                        '''
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
                    docker.build("${DOCKER_USER}/microservices-${APP_NAME}:${IMAGE_TAG}")
                }
            }
        }

        stage("Trivy Image Scan") {
            steps {
                script {
                    // Perform Trivy image scan
                    sh "docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image ${DOCKER_USER}/microservices-${APP_NAME}:${IMAGE_TAG} --no-progress --exit-code 0 --severity HIGH,CRITICAL --format table --scanners vuln --timeout 50m | tee trivy_${APP_NAME}.txt"
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_PASS) {
                        // Push the Docker image to Docker Hub
                        docker.image("${DOCKER_USER}/microservices-${APP_NAME}:${IMAGE_TAG}").push()
                    }
                }
            }
        }

        stage('Cleanup Artifact') {
            steps {
                script {
                    // Remove Docker image
                    sh "docker rmi ${DOCKER_USER}/microservices-${APP_NAME}:${IMAGE_TAG}"
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: "trivy_${APP_NAME}.txt", allowEmptyArchive: true
        }
    }
}
