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
        stage('Deploy Wild West K8'){
           steps{
               sh '''
                  kubectl apply -f https://git.io/k8s-wild-west
                  kubectl apply -f https://git.io/k8s-wild-west-destructive
                  kubectl port-forward -n wildwest svc/wildwest 9090:8080
                  '''
           }
        }
        //  stage('Stop Minikube') {
        //     steps {
        //         sh("minikube delete")
        //     }
        // }

    }
   
}