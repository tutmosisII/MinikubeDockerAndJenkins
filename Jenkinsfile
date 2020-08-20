pipeline{
    agent{
        label 'localms'
    }
    stages {
        stage('Start Minikube') {
            steps {
                // Create namespace if it doesn't exist
                sh("minikube start")
            }
        }

    }
   
}