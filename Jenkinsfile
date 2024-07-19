pipeline {
    agent any
    
    environment {
        REGISTRY = "docker.io/my-org"
        IMAGE = "myapp"
        K8S_CLUSTER = "kubeconfig"
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/my-org/myapp.git'
            }
        }
        
        stage('Build') {
            steps {
                script {
                    def image = docker.build("${env.REGISTRY}/${env.IMAGE}:${env.BUILD_ID}")
                }
            }
        }
        
        stage('Push') {
            steps {
                script {
                    docker.withRegistry("https://${env.REGISTRY}", 'docker_credentials') {
                        image.push()
                    }
                }
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    def currentDeployment = sh(script: "kubectl get svc myapp-service -o jsonpath='{.spec.selector.app}'", returnStdout: true).trim()
                    def newDeployment = currentDeployment == 'blue' ? 'green' : 'blue'
                    
                    // Deploy to the new environment
                    sh "kubectl apply -f k8s/${newDeployment}-deployment.yaml"
                    
                    // Switch the service to the new deployment
                    sh "kubectl patch svc myapp-service -p '{\"spec\":{\"selector\":{\"app\":\"${newDeployment}\"}}}'"
                }
            }
        }
    }
    
    post {
        failure {
            script {
                def failedDeployment = sh(script: "kubectl get svc myapp-service -o jsonpath='{.spec.selector.app}'", returnStdout: true).trim()
                def previousDeployment = failedDeployment == 'blue' ? 'green' : 'blue'
                
                // Revert the service to the previous deployment
                sh "kubectl patch svc myapp-service -p '{\"spec\":{\"selector\":{\"app\":\"${previousDeployment}\"}}}'"
                
                // Optionally, delete the failed deployment
                sh "kubectl delete deployment ${failedDeployment}-deployment"
            }
        }
    }
}
