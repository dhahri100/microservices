pipeline {
    agent any

    environment {
        CREDENTIALS_ID = '85215780-f231-45ec-a1c5-304a7cd774ff'
        PROJECT_ID = 'calm-photon-436315-j1'
        CLUSTER_NAME = 'gke'
        LOCATION = 'us-central1-c'
        NAMESPACE = 'prod' 
    }

    stages {

        stage('Deploy to K8s') {
		    steps{
			    echo "Deployment started ..."
			    step([$class: 'KubernetesEngineBuilder', projectId: env.PROJECT_ID, clusterName: env.CLUSTER_NAME, location: env.LOCATION, manifestPattern: 'fulldeployment.yaml', credentialsId: env.CREDENTIALS_ID, verifyDeployments: true,namespace: env.NAMESPACE])
				echo "Start deployment of deployment.yaml"
				
			    echo "Deployment Finished ..."
		    }
	    }
    }
}
