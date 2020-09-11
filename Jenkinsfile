pipeline{
    agent{
        label 'localms'
    }
    stages {
        stage('Start Minikube') {
            steps {
                sh("minikube start")
            }
        }
         stage('Stop Minikube') {
            steps {
                sh("minikube delete")
            }
        }

    }
   
}