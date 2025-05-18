pipeline {
    agent any

    environment {
        SCANNER_HOME = tool name: 'SonarQube-Scanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'  // Configured SonarQube Scanner
        SONAR_PROJECT_KEY = "microservices-checkoutservice"  // Specify project key
        SONAR_PROJECT_NAME = "Checkout Service"  // Optional: Give the project a name
        SONAR_PROJECT_VERSION = "1.0"
        APP_NAME = "checkoutservice"  // The name of the Docker image
        RELEASE = "1.0.0"
        DOCKER_USER = "mouhib19"  // DockerHub username
        DOCKER_PASS = "dockerhub"  // DockerHub password
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"  // Tag for the image
    }

    stages {
        
          stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv('SonarQube-Server') {
                    sh '''
                        ${SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=checkoutservice \
                        -Dsonar.projectName="Checkout Service" \
                        -Dsonar.projectVersion=1.0 \
                        -Dsonar.sources=.
                    '''
                    }

                }
            }
        }

        /*stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true  // Wait for SonarQube quality gate
                }
            }
        }*/
        stage('OWASP Dependency-Check Vulnerabilities') {
                      steps {
                        dependencyCheck additionalArguments: ''' 
                                    -o './'
                                    -s './'
                                    -f 'ALL' 
                                    --prettyPrint''', odcInstallation: 'Owasp'
                        
                        dependencyCheckPublisher pattern: 'dependency-check-report.xml'
                      }
                }

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
        stage('Deploy to Minikube') {
                        steps {
                       sh '''
                        export KUBECONFIG=/home/jenkins/.kube/config
                        kubectl config use-context minikube
                        kubectl apply -f checkoutservice.yaml
                    '''
                        }
                    }
    }

    post {
        always {
            archiveArtifacts artifacts: "trivy_${APP_NAME}.txt", allowEmptyArchive: true  // Archive Trivy scan results
        }
    }
}
