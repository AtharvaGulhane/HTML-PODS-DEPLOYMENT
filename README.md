"# HTML-PODS-DEPLOYMENT" 

**1. Create an HTML File**

- **File:** index.html

<!DOCTYPE html>

<html>

<head>

`    `<title>My HTML in Docker</title>

</head>

<body>

`    `<h1>Hello, World!</h1>

`    `<p>This HTML is running inside a Docker container!</p>

</body>

</html>

- This HTML file is the content that will be served by the Nginx web server inside a Docker container.

**2. Create a Dockerfile**

- **File:** Dockerfile

\# Use an official Nginx image as the base

FROM nginx:alpine

\# Copy the HTML file into the container

COPY  index.html /usr/share/nginx/html/index.html

\# Expose port 80 to the outside world

EXPOSE 80

- This Dockerfile sets up an Nginx server in a Docker container that serves the index.html file.

**

**3. Build and Push the Docker Image**

- **Build the Docker Image:**

  docker build -t gulhaneatharva/demo-html-proj .

- **Push the Docker Image to Docker Hub:**

  docker push gulhaneatharva/demo-html-proj

- This step creates a Docker image with your HTML file served by Nginx and uploads it to Docker Hub.

**4. Create Kubernetes YAML Files**

- **Pod YAML File (my-html-pod.yaml):**

apiVersion: v1

kind: Pod

metadata:

`  `name: my-html-pod

`  `labels:

`    `app: my-html-app

spec:

`  `containers:

`  `- name: demo-html-container

`    `image: gulhaneatharva/demo-html-proj

`    `ports:

`    `- containerPort: 80



- **Service YAML File (my-html-service.yaml):**

apiVersion: v1

kind: Service

metadata:

`  `name: my-html-service

spec:

`  `selector:

`    `app: my-html-app

`  `ports:

`  `- protocol: TCP

`    `port: 80

`    `targetPort: 80

`  `type: NodePort

- These YAML files define a Pod that runs the Docker container and a Service that exposes the Pod on a specific port.



**5. Apply the Kubernetes Configuration**

- **Deploy the Pod:**

  kubectl apply -f my-html-pod.yaml

- **Deploy the Service:**

  kubectl apply -f my-html-service.yaml

- This step creates the Pod and the Service in your Kubernetes cluster.

**6. Verify the Deployment**

- **Check Pod and Service Status:**

kubectl get pods

kubectl get services

- **Check Logs of the Pod:**

  kubectl logs my-html-pod

- **Test Service Internally:**

  kubectl exec -it my-html-pod -- curl <http://my-html-service:80>

- This verifies that the Pod is running and the Service is correctly routing traffic to it.

**7. Expose and Access the Service**

- **Expose the Service via Minikube:**

  minikube service my-html-service



- **Access the Service via Browser:**
  - After running the above command, you were given a URL like http://127.0.0.1:53968 to access the service locally through your web browser.

- This step allowed you to expose the service on your local machine and access the web page served by Nginx.

**8. Set Up Jenkins Pipeline**

- **SCM Configuration:**
  - You set up the Jenkins pipeline using an SCM-based Jenkinsfile, which pulls the configuration from a GitHub repository.
  - GitHub credentials (github-private-repo) were configured in Jenkins for secure access to the repository.
  - GIT REPO - <https://github.com/AtharvaGulhane/HTML-PODS-DEPLOYMENT>
  - JENKINS FILE - <https://github.com/AtharvaGulhane/HTML-PODS-DEPLOYMENT/blob/main/Jenkinsfile>

pipeline {

`    `agent any

`    `environment {

`        `DOCKER\_IMAGE = 'gulhaneatharva/demo-html-proj'

`        `DOCKERFILE\_PATH = './Dockerfile'

`        `DOCKER\_CREDENTIALS\_ID = 'dockerhub-credentials'

`        `GIT\_CREDENTIALS\_ID = 'github-private-repo'

`        `KUBECONFIG\_PATH = 'C:\\Users\\AG\\.kube\\config'

`    `}

`    `stages {

`        `stage('Checkout Code') {

`            `steps {

`                `git credentialsId: "${GIT\_CREDENTIALS\_ID}", url: 'https://github.com/AtharvaGulhane/HTML-PODS-DEPLOYMENT.git', branch: 'main'

`            `}

`        `}

`        `stage('Build Docker Image') {

`            `steps {

`                `script {

`                    `def buildImage = docker.build("${DOCKER\_IMAGE}:${BUILD\_NUMBER}", "-f ${DOCKERFILE\_PATH} .")

`                `}

`            `}

`        `}

`        `stage('Update Docker Image') {

`            `steps {

`                `script {

`                    `docker.withRegistry('https://index.docker.io/v1/', "${DOCKER\_CREDENTIALS\_ID}") {

`                        `def image = docker.image("${DOCKER\_IMAGE}:${BUILD\_NUMBER}")

`                        `image.push()

`                        `image.push('latest')  // Push the "latest" tag as well

`                    `}

`                `}

`            `}

`        `}

`        `stage('Deploy to Minikube') {

`            `steps {

`                `script {

`                    `def kubectlCmd = 'kubectl'

`                    `if (isUnix()) {

`                        `// Unix-like environments

`                        `sh "${kubectlCmd} config use-context minikube"

`                        `sh "${kubectlCmd} apply -f my-html-pod.yaml"

`                        `sh "${kubectlCmd} apply -f my-html-service.yaml"

`                    `} else {

`                        `// Windows environment using cmd or PowerShell

`                        `bat "${kubectlCmd} config use-context minikube"

`                        `bat "${kubectlCmd} apply -f my-html-pod.yaml"

`                        `bat "${kubectlCmd} apply -f my-html-service.yaml"

`                    `}

`                `}

`            `}

`        `}

`        `stage('Verify Deployment') {

`            `steps {

`                `script {

`                    `if (isUnix()) {

`                        `sh 'kubectl get pods'

`                        `sh 'kubectl get services'

`                    `} else {

`                        `bat 'kubectl get pods'

`                        `bat 'kubectl get services'

`                    `}

`                `}

`            `}

`        `}

`    `}

`    `post {

`        `success {

`            `echo 'Pipeline succeeded!'

`        `}

`        `failure {

`            `echo 'Pipeline failed!'

`        `}

`        `always {

`            `cleanWs()  // Clean up the workspace after the pipeline completes

`        `}

`    `}

}

**9. Pipeline Stages**

- **Checkout Code:**
  - The pipeline starts by checking out the latest code from the GitHub repository using the git step.
  - The GIT\_CREDENTIALS\_ID environment variable is used to provide authentication.
- **Build Docker Image:**
  - In this stage, Jenkins builds a Docker image from the project's Dockerfile.
    - DOCKER FILE –  <https://github.com/AtharvaGulhane/HTML-PODS-DEPLOYMENT/blob/main/Dockerfile>
  - The image is tagged with the build number for versioning purposes.
- **Update Docker Image:**
  - The built Docker image is pushed to Docker Hub, using the credentials provided in the DOCKER\_CREDENTIALS\_ID environment variable.
  - The image is pushed both with a version tag (using the build number) and with the latest tag.
- **Deploy to Minikube:**
  - The pipeline detects whether it's running on a Windows or Unix-based system.
  - For Windows, the pipeline uses cmd or PowerShell to execute kubectl commands directly, while Unix systems use standard shell commands.
  - Kubernetes configuration files ([my-html-pod.yaml](https://github.com/AtharvaGulhane/HTML-PODS-DEPLOYMENT/blob/main/my-html-pod.yaml) and [my-html-service.yaml](https://github.com/AtharvaGulhane/HTML-PODS-DEPLOYMENT/blob/main/my-html-service.yaml)) are applied to Minikube to deploy the application.
- **Verify Deployment:**
  - After deployment, the pipeline verifies the deployment by listing the pods and services in the Kubernetes cluster.

    And You can manually check - minikube service my-html-service

    |-----------|-----------------|-------------|---------------------------|

    | NAMESPACE |      NAME       | TARGET PORT |            URL            |

    |-----------|-----------------|-------------|---------------------------|

    | default   | my-html-service |          80 | http://192.168.49.2:32243 |

    |-----------|-----------------|-------------|---------------------------|

    🏃  Starting tunnel for service my-html-service.

    |-----------|-----------------|-------------|------------------------|

    | NAMESPACE |      NAME       | TARGET PORT |          URL           |

    |-----------|-----------------|-------------|------------------------|

    | default   | my-html-service |             | http://127.0.0.1:64563 |

    |-----------|-----------------|-------------|------------------------|

    🎉  Opening service default/my-html-service in default browser...

    ❗  Because you are using a Docker driver on windows, the terminal needs to be open to run it.

    ✋  Stopping tunnel for service my-html-service.

  - This ensures that the application is running as expected.

**10. Post-Execution Actions**

- **Success/Failure Notifications:**
  - The pipeline prints a success or failure message depending on the outcome of the stages.
  - If the pipeline fails at any stage, this information is logged for easier debugging.
- **Workspace Cleanup:**
  - The pipeline uses cleanWs() to clean up the workspace, ensuring that no residual files are left behind after the pipeline completes.

**4. Environment Variables**

- **DOCKER\_IMAGE:**
  - Defines the name of the Docker image that is built and pushed.
- **DOCKERFILE\_PATH:**
  - Specifies the path to the Dockerfile used to build the image.
- **DOCKER\_CREDENTIALS\_ID:**
  - Stores Docker Hub credentials for authentication.
- **GIT\_CREDENTIALS\_ID:**
  - Stores GitHub credentials for secure access to the repository.
- **KUBECONFIG\_PATH:**
  - Defines the path to the Kubeconfig file, allowing Jenkins to interact with Minikube.


