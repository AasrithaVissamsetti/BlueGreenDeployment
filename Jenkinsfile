pipeline {
    agent any

    stages {
        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Deploy Blue') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-cred', variable: 'KUBECONFIG')]) {
                    sh 'kubectl apply -f deployment-blue.yaml'
                    sh 'kubectl apply -f service.yaml'
                }
            }
        }

        stage('Deploy Green') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-cred', variable: 'KUBECONFIG')]) {
                    sh 'kubectl apply -f deployment-green.yaml'
                }
            }
        }

        stage('Test Green') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-cred', variable: 'KUBECONFIG')]) {
                    sh '''
                        kubectl port-forward deployment/myapp-green 8080:80 &
                        sleep 10
                        curl -f http://localhost:8080
                    '''
                }
            }
        }

        stage('Switch Traffic to Green') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-cred', variable: 'KUBECONFIG')]) {
                    sh '''
                        kubectl patch service myapp-service \
                          -p '{"spec":{"selector":{"app":"myapp","version":"green"}}}'
                    '''
                }
            }
        }

        stage('Verify Switch') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-cred', variable: 'KUBECONFIG')]) {
                    sh 'kubectl get svc myapp-service -o yaml'
                }
            }
        }

        stage('Cleanup Blue (Optional)') {
            when {
                expression { return params.CLEANUP_BLUE == true }
            }
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-cred', variable: 'KUBECONFIG')]) {
                    sh 'kubectl delete deployment myapp-blue || true'
                }
            }
        }
    }
}
