pipeline {
    agent any

    environment {
        // Path inside the Jenkins container where Minikube config is mounted
        KUBECONFIG = '/var/jenkins_home/.minikube/config'
    }

    parameters {
        booleanParam(name: 'CLEANUP_BLUE', defaultValue: true, description: 'Cleanup old blue deployment after switch?')
    }

    stages {
        stage('Checkout SCM') {
            steps {
                // Checkout the Git repository
                checkout scm
            }
        }

        stage('Deploy Blue') {
            steps {
                withEnv(["KUBECONFIG=${env.KUBECONFIG}"]) {
                    sh 'kubectl apply -f deployment-blue.yaml'
                    sh 'kubectl apply -f service.yaml'
                }
            }
        }

        stage('Deploy Green') {
            steps {
                withEnv(["KUBECONFIG=${env.KUBECONFIG}"]) {
                    sh 'kubectl apply -f deployment-green.yaml'
                }
            }
        }

        stage('Test Green') {
            steps {
                withEnv(["KUBECONFIG=${env.KUBECONFIG}"]) {
                    // Port-forward for testing; adjust as needed
                    sh 'kubectl port-forward deployment/myapp-green 8080:80 & sleep 10'
                    sh 'curl -f http://localhost:8080'
                }
            }
        }

        stage('Switch Traffic to Green') {
            steps {
                withEnv(["KUBECONFIG=${env.KUBECONFIG}"]) {
                    sh '''kubectl patch service myapp-service \
                        -p '{"spec":{"selector":{"app":"myapp","version":"green"}}}' '''
                }
            }
        }

        stage('Verify Switch') {
            steps {
                withEnv(["KUBECONFIG=${env.KUBECONFIG}"]) {
                    sh 'kubectl get svc myapp-service -o yaml'
                }
            }
        }

        stage('Cleanup Blue (Optional)') {
            when {
                expression { return params.CLEANUP_BLUE == true }
            }
            steps {
                withEnv(["KUBECONFIG=${env.KUBECONFIG}"]) {
                    sh 'kubectl delete deployment myapp-blue || true'
                }
            }
        }
    }
}
