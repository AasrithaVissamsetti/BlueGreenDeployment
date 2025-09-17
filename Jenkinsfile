pipeline {
    agent any

    environment {
        // Correct kubeconfig file path inside the container
        KUBECONFIG = '/var/jenkins_home/.minikube/profiles/minikube/config'
    }

    stages {
        stage('Checkout SCM') {
            steps {
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
