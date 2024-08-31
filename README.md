# HTML-PODS-DEPLOYMENT

This project demonstrates deploying a simple HTML page inside a Docker container using Kubernetes and Jenkins.

---

## 1. Create an HTML File

- **File:** `index.html`

```html
<!DOCTYPE html>
<html>
<head>
    <title>My HTML in Docker</title>
</head>
<body>
    <h1>Hello, World!</h1>
    <p>This HTML is running inside a Docker container!</p>
</body>
</html>
```

- This HTML file will be served by the Nginx web server inside a Docker container.

---

## 2. Create a Dockerfile

- **File:** `Dockerfile`

```dockerfile
# Use an official Nginx image as the base
FROM nginx:alpine

# Copy the HTML file into the container
COPY index.html /usr/share/nginx/html/index.html

# Expose port 80 to the outside world
EXPOSE 80
```

- This Dockerfile sets up an Nginx server that serves the `index.html` file.

---

## 3. Build and Push the Docker Image

### Build the Docker Image:

```bash
docker build -t gulhaneatharva/demo-html-proj .
```

### Push the Docker Image to Docker Hub:

```bash
docker push gulhaneatharva/demo-html-proj
```

- These commands build a Docker image with your HTML file and upload it to Docker Hub.

---

## 4. Create Kubernetes YAML Files

### Pod YAML File (`my-html-pod.yaml`):

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-html-pod
  labels:
    app: my-html-app
spec:
  containers:
  - name: demo-html-container
    image: gulhaneatharva/demo-html-proj
    ports:
    - containerPort: 80
```

### Service YAML File (`my-html-service.yaml`):

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-html-service
spec:
  selector:
    app: my-html-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: NodePort
```

- These YAML files define a Pod that runs the Docker container and a Service that exposes the Pod on a specific port.

---

## 5. Apply the Kubernetes Configuration

### Deploy the Pod:

```bash
kubectl apply -f my-html-pod.yaml
```

### Deploy the Service:

```bash
kubectl apply -f my-html-service.yaml
```

- These commands create the Pod and Service in your Kubernetes cluster.

---

## 6. Verify the Deployment

### Check Pod and Service Status:

```bash
kubectl get pods
kubectl get services
```

### Check Logs of the Pod:

```bash
kubectl logs my-html-pod
```

### Test Service Internally:

```bash
kubectl exec -it my-html-pod -- curl http://my-html-service:80
```

- These steps verify that the Pod is running and the Service is correctly routing traffic to it.

---

## 7. Expose and Access the Service

### Expose the Service via Minikube:

```bash
minikube service my-html-service
```

### Access the Service via Browser:

- After running the above command, you will receive a URL like `http://127.0.0.1:53968` to access the service locally through your web browser.

---

## 8. Set Up Jenkins Pipeline

### SCM Configuration:

- You set up the Jenkins pipeline using an SCM-based Jenkinsfile, which pulls the configuration from a GitHub repository.
- GitHub credentials (`github-private-repo`) were configured in Jenkins for secure access to the repository.

**GitHub Repository:** [HTML-PODS-DEPLOYMENT](https://github.com/AtharvaGulhane/HTML-PODS-DEPLOYMENT)

**Jenkinsfile Location:** [Jenkinsfile](https://github.com/AtharvaGulhane/HTML-PODS-DEPLOYMENT/blob/main/Jenkinsfile)

### Jenkins Pipeline Definition:

```groovy
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
                    def kubectlCmd = 'kubectl'
                    if (isUnix()) {
                        // Unix-like environments
                        sh "${kubectlCmd} config use-context minikube"
                        sh "${kubectlCmd} apply -f my-html-pod.yaml"
                        sh "${kubectlCmd} apply -f my-html-service.yaml"
                    } else {
                        // Windows environment using cmd or PowerShell
                        bat "${kubectlCmd} config use-context minikube"
                        bat "${kubectlCmd} apply -f my-html-pod.yaml"
                        bat "${kubectlCmd} apply -f my-html-service.yaml"
                    }
                }
            }
        }
        stage('Verify Deployment') {
            steps {
                script {
                    if (isUnix()) {
                        sh 'kubectl get pods'
                        sh 'kubectl get services'
                    } else {
                        bat 'kubectl get pods'
                        bat 'kubectl get services'
                    }
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
```

---

## 9. Pipeline Stages

### **1. Checkout Code:**
  - The pipeline checks out the latest code from the GitHub repository using the `git` step.

### **2. Build Docker Image:**
  - Jenkins builds a Docker image from the project's Dockerfile, tagging it with the build number for versioning.

### **3. Update Docker Image:**
  - The built Docker image is pushed to Docker Hub, both with a version tag (using the build number) and with the `latest` tag.

### **4. Deploy to Minikube:**
  - The pipeline detects whether it's running on a Windows or Unix-based system and executes `kubectl` commands accordingly.

### **5. Verify Deployment:**
  - The pipeline verifies the deployment by listing the pods and services in the Kubernetes cluster.

### **Manual Verification:**
  - You can manually verify the service by running `minikube service my-html-service`.

---

## 10. Post-Execution Actions

### **Success/Failure Notifications:**
  - The pipeline logs a success or failure message, making it easier to debug any issues.

### **Workspace Cleanup:**
  - The pipeline uses `cleanWs()` to ensure that no residual files are left behind after the pipeline completes.

---

## 11. Environment Variables

- **DOCKER_IMAGE:** Defines the name of the Docker image that is built and pushed.
- **DOCKERFILE_PATH:** Specifies the path to the Dockerfile used to build the image.
- **DOCKER_CREDENTIALS_ID:** Stores Docker Hub credentials for authentication.
- **GIT_CREDENTIALS_ID:** Stores GitHub credentials for secure access to the repository.
- **KUBECONFIG_PATH:** Defines the path to the Kubeconfig file, allowing Jenkins to interact with Minikube.

---
