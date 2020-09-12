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
               timeout(10) {
                    waitUntil {
                        script {
                            def r = sh script: 'kubectl port-forward -n wildwest svc/wildwest 9090:8080', returnStdout: true
                            return (r == 0);
                        }
                    }
              }               
           }
        }
        //  stage('Stop Minikube') {
        //     steps {
        //         sh("minikube delete")
        //     }
        // }

    }
   
}