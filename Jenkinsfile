pipeline {
    agent any

    environment {
        KUBECONFIG = credentials('kubeconfig-cred') // Jenkins credential with kubeconfig
    }

    stages {
        stage('Deploy Blue') {
            steps {
                sh 'kubectl apply -f deployment-blue.yaml'
                sh 'kubectl apply -f service.yaml'
            }
        }

        stage('Deploy Green') {
            steps {
                sh 'kubectl apply -f deployment-green.yaml'
            }
        }

        stage('Test Green') {
            steps {
                // Example: Port-forward and test
                // Replace with curl tests, integration tests, etc.
                sh 'kubectl port-forward deployment/myapp-green 8080:80 & sleep 10'
                sh 'curl -f http://localhost:8080'
            }
        }

        stage('Switch Traffic to Green') {
            steps {
                sh '''kubectl patch service myapp-service \
                    -p '{"spec":{"selector":{"app":"myapp","version":"green"}}}' '''
            }
        }

        stage('Verify Switch') {
            steps {
                sh 'kubectl get svc myapp-service -o yaml'
            }
        }

        stage('Cleanup Blue (Optional)') {
            when {
                expression { return params.CLEANUP_BLUE == true }
            }
            steps {
                sh 'kubectl delete deployment myapp-blue || true'
            }
        }
    }
}
