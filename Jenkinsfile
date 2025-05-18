pipeline {
    agent any

    environment {
        SCANNER_HOME = tool name: 'SonarQube-Scanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
        SONAR_PROJECT_KEY = "cartservice"  // Specify your project key here
        SONAR_PROJECT_NAME = "Cart Service"  // Optional: Give your project a name
        SONAR_PROJECT_VERSION = "1.0"
        APP_NAME = "cartservice"  // The name of the image (cartservice)
        RELEASE = "1.0.0"
        DOCKER_USER = "mouhib19"  // Your DockerHub username
        DOCKER_PASS = "dockerhub"  
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
    }
    
    stages {

       /* stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv('SonarQube-Server') {
                    sh '''
                        ${SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=cartservice \
                        -Dsonar.projectName="Cart Service" \
                        -Dsonar.projectVersion=1.0 \
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
        
        /*stage('OWASP Dependency-Check Vulnerabilities') {
                      steps {
                        dependencyCheck additionalArguments: ''' 
                                    -o './'
                                    -s './'
                                    -f 'ALL' 
                                    --prettyPrint''', odcInstallation: 'Owasp'
                        
                        dependencyCheckPublisher pattern: 'dependency-check-report.xml'
                      }
                }
        */
        stage('Build Docker Image') {
            steps {
                script {
                    dir('src') { 
                        docker.build("${DOCKER_USER}/microservices-${APP_NAME}:${IMAGE_TAG}")
                    }
                }
            }
        }
        stage("Trivy Image Scan") {
            steps {
                script {
                    sh "docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image ${DOCKER_USER}/microservices-${APP_NAME}:${IMAGE_TAG} --no-progress --exit-code 0 --severity HIGH,CRITICAL --format table --scanners vuln --timeout 50m > trivy_${APP_NAME}.txt"
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    
                        docker.withRegistry('https://index.docker.io/v1/', DOCKER_PASS) {
                            docker.image("${DOCKER_USER}/microservices-${APP_NAME}:${IMAGE_TAG}").push()
                        }
                    }
                }
            
}


        stage ('Cleanup Artifact') {
            steps {
                script {
                    sh "docker rmi ${DOCKER_USER}/microservices-${APP_NAME}:${IMAGE_TAG}"  // Remove Docker image
                }
            }
        }
       /* stage('Deploy to Minikube') {
                        steps {
                       sh '''
                        export KUBECONFIG=/home/jenkins/.kube/config
                        kubectl config use-context minikube
                        kubectl apply -f cartservice.yaml
                    '''
                        }
                    }*/
    }

    post {
        always {
            archiveArtifacts artifacts: "trivy_${APP_NAME}.txt", allowEmptyArchive: true
        }
    }
}
