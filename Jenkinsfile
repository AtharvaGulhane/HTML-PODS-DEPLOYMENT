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
                git credentialsId: "${GIT_CREDENTIALS_ID}", url: 'https://github.com/AtharvaGulhane/HTML-PODS-DEPLOYMENT.git', branch: 'main'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def buildImage = docker.build("${DOCKER_IMAGE}:${BUILD_NUMBER}", "-f ${DOCKERFILE_PATH} .")
                }
            }
        }

        stage('Update Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_CREDENTIALS_ID}") {
                        def image = docker.image("${DOCKER_IMAGE}:${BUILD_NUMBER}")
                        image.push()
                        image.push('latest')  // Push the "latest" tag as well
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
                    sh 'kubectl get pods'
                    sh 'kubectl get services'
                    // Optional: Check the status of the deployment rollout
                    // sh 'kubectl rollout status deployment/my-html-deployment'
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
        always {
            cleanWs()  // Clean up the workspace after the pipeline completes
        }
    }
}
