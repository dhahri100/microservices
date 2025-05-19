pipeline {
    agent any


    stages {
   stage('Deploy to Minikube') {
                        steps {
                       sh '''
                        export KUBECONFIG=/home/jenkins/.kube/config
                        kubectl config use-context minikube
                        kubectl apply -f fulldeployment.yaml
                    '''
                        }
                    }
}
}
