pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'dockerhub-credentials'
        GIT_CREDENTIALS_ID = 'github-private-repo'
        KUBECONFIG_PATH = 'C:\\Users\\AG\\.kube\\config'
        IMAGE_NAME = 'gulhaneatharva/demo-html-proj'
    }

    stages {
        stage('Checkout Code') {
            steps {
                // Checkout the code from the private GitHub repository
                git credentialsId: "${GIT_CREDENTIALS_ID}", url: 'https://github.com/AtharvaGulhane/HTML-PODS-DEPLOYMENT.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image
                    sh 'docker build -t ${IMAGE_NAME} .'
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    // Log in to Docker Hub and push the Docker image
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_CREDENTIALS_ID}") {
                        sh 'docker push ${IMAGE_NAME}'
                    }
                }
            }
        }

        stage('Deploy to Minikube') {
            steps {
                script {
                    // Set up kubeconfig environment variable
                    withKubeConfig([credentialsId: "${KUBECONFIG_PATH}"]) {
                        // Apply Kubernetes configuration
                        sh 'kubectl apply -f my-html-pod.yaml'
                        sh 'kubectl apply -f my-html-service.yaml'
                    }
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                script {
                    // Check the status of the pod and service
                    sh 'kubectl get pods'
                    sh 'kubectl get services'
                }
            }
        }
    }

    post {
        failure {
            // Handle failure scenarios
            echo 'Build or Deployment failed!'
        }
        success {
            // Handle success scenarios
            echo 'Build and Deployment succeeded!'
        }
    }
}
