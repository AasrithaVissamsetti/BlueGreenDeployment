pipeline {
    agent any

    environment {
        KUBECONFIG = credentials('kubeconfig-cred') // Jenkins credential with kubeconfig
    }

    stages {
        stage('Deploy Blue') {
            steps {
                bat 'kubectl apply -f deployment-blue.yaml'
                bat 'kubectl apply -f service.yaml'
            }
        }

        stage('Deploy Green') {
            steps {
                bat 'kubectl apply -f deployment-green.yaml'
            }
        }

        stage('Test Green') {
            steps {
                // Example: Port-forward and test
                // Replace with curl tests, integration tests, etc.
                bat 'kubectl port-forward deployment/myapp-green 8080:80 & sleep 10'
                bat 'curl -f http://localhost:8080'
            }
        }

        stage('Switch Traffic to Green') {
            steps {
                bat '''kubectl patch service myapp-service \
                    -p '{"spec":{"selector":{"app":"myapp","version":"green"}}}' '''
            }
        }

        stage('Verify Switch') {
            steps {
                bat 'kubectl get svc myapp-service -o yaml'
            }
        }

        stage('Cleanup Blue (Optional)') {
            when {
                expression { return params.CLEANUP_BLUE == true }
            }
            steps {
                bat 'kubectl delete deployment myapp-blue || true'
            }
        }
    }
}
