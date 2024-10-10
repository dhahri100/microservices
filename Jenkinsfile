pipeline {
    agent any

    environment {
        SCANNER_HOME = tool name: 'SonarQube-Scanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
        SONAR_PROJECT_KEY = "shippingservice"  // Specify your project key here
        SONAR_PROJECT_NAME = "Shipping Service"  // Optional: Give your project a name
        SONAR_PROJECT_VERSION = "1.0"
        APP_NAME = "shippingservice"
        RELEASE = "1.0.0"
        DOCKER_USER = "mouhib543"
        DOCKER_PASS = "dockerhub"  
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
    }
    
    stages {

        /*stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv('SonarQube-Server') {
                    sh '''
                        /var/lib/jenkins/tools/hudson.plugins.sonar.SonarRunnerInstallation/SonarQube-Scanner/bin/sonar-scanner \
                        -Dsonar.projectKey=shippingservice \
                        -Dsonar.projectName="Shipping Service" \
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

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_USER}/${APP_NAME}:${IMAGE_TAG}")
                }
            }
        }

        /*stage("Trivy Image Scan") {
            steps {
                script {
                    sh "docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image ${DOCKER_USER}/${APP_NAME}:${IMAGE_TAG} --no-progress --exit-code 0 --severity HIGH,CRITICAL --format table --scanners vuln --timeout 50m | tee trivy_${APP_NAME}.txt"
                }
            }
        }*/

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_PASS) {
                        docker.image("${DOCKER_USER}/${APP_NAME}:${IMAGE_TAG}").push()
                    }
                }
            }
        }

        stage ('Cleanup Artifact') {
            steps {
                script {
                    sh "docker rmi ${DOCKER_USER}/${APP_NAME}:${IMAGE_TAG}"
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
