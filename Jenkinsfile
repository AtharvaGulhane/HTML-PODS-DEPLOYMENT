pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'gulhaneatharva/demo-html-proj'
        DOCKERFILE_PATH = './Dockerfile'
        DOCKER_CREDENTIALS_ID = 'dockerhub-credentials'
        GIT_CREDENTIALS_ID = 'github-private-repo'
        KUBECONFIG_PATH = 'C:\\Users\\AG\\.kube\\config'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git credentialsId: "${GIT_CREDENTIALS_ID}", url: 'https://github.com/AtharvaGulhane/HTML-PODS-DEPLOYMENT.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image
                    def buildImage = docker.build("${DOCKER_IMAGE}", "-f ${DOCKERFILE_PATH} .")
                }
            }
        }

        stage('Update Docker Image') {
            steps {
                script {
                    // Log in to Docker Hub and push the Docker image
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_CREDENTIALS_ID}") {
                        def image = docker.image("${DOCKER_IMAGE}")
                        image.push('latest')
                    }
                }
            }
        }

        stage('Deploy to Minikube') {
            steps {
                script {
                    // Set up kubectl context to use Minikube
                    sh 'kubectl config use-context minikube'

                    // Apply Kubernetes configuration files
                    sh 'kubectl apply -f my-html-pod.yaml'
                    sh 'kubectl apply -f my-html-service.yaml'
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
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
