Certainly! Here's a template you can use to create a detailed **README.md** file for your project. I'll include sections for **setting up the environment**, **deploying the application**, and **explaining the Terraform code and Jenkins pipeline**.

---

# **Hello World Application - EKS Deployment**

This repository contains a simple Flask application that displays "Hello, World!". The application is Dockerized and deployed on an AWS EKS cluster. CI/CD is handled by Jenkins, with images being built using Docker and stored in AWS ECR (Elastic Container Registry).

## **Table of Contents**

1. [Project Overview](#project-overview)
2. [Setting Up the Environment](#setting-up-the-environment)
3. [Deploying the Application](#deploying-the-application)
4. [Terraform Code Explanation](#terraform-code-explanation)
5. [Jenkins Pipeline Explanation](#jenkins-pipeline-explanation)

---

## **Project Overview**

- **Programming Language**: Python (Flask)
- **Containerization**: Docker
- **Orchestration**: Kubernetes on AWS EKS
- **CI/CD**: Jenkins
- **Registry**: AWS ECR

---

## **Setting Up the Environment**

### **Prerequisites**

1. **AWS CLI** installed and configured:
    - Follow the instructions [here](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html) to install the AWS CLI.
    - Run `aws configure` and set your AWS credentials.
   
2. **Docker** installed:
    - Follow instructions [here](https://docs.docker.com/get-docker/) to install Docker.

3. **Terraform** installed:
    - Download and install Terraform from [here](https://learn.hashicorp.com/tutorials/terraform/install-cli).

4. **Kubectl** installed:
    - Install `kubectl` by following the guide [here](https://kubernetes.io/docs/tasks/tools/).

5. **Jenkins** running on an EC2 instance or locally.

---

## **Deploying the Application**

### **Step 1: Clone the Repository**

```bash
git clone https://github.com/sammikenzie/terraform-eks.git
cd terraform-eks
```

### **Step 2: Provision AWS Resources Using Terraform**

1. Initialize Terraform:
   ```bash
   terraform init
   ```

2. Create the EKS cluster and associated resources:
   ```bash
   terraform apply
   ```

   - This will create an AWS VPC, subnets, security groups, and an EKS cluster.

3. After Terraform has successfully created the infrastructure, it will output the EKS cluster details. Update your local `kubectl` config to point to the new cluster:
   ```bash
   aws eks update-kubeconfig --region us-east-2 --name my-eks-cluster
   ```

### **Step 3: Build and Push Docker Image**

1. Build the Docker image:
   ```bash
   docker build -t hello-world-app .
   ```

2. Tag the Docker image for AWS ECR:
   ```bash
   docker tag hello-world-app:latest <AWS_ACCOUNT_ID>.dkr.ecr.<AWS_REGION>.amazonaws.com/hello-world-app:latest
   ```

3. Push the Docker image to ECR:
   ```bash
   docker push <AWS_ACCOUNT_ID>.dkr.ecr.<AWS_REGION>.amazonaws.com/hello-world-app:latest
   ```

### **Step 4: Deploy the Application to EKS**

1. Apply the Kubernetes `deployment.yaml` and `service.yaml` files to deploy the application:
   ```bash
   kubectl apply -f deployment.yaml
   kubectl apply -f service.yaml
   ```

2. Check the status of the pods:
   ```bash
   kubectl get pods
   ```

3. Get the external IP of the service:
   ```bash
   kubectl get svc
   ```

4. Visit the application in your browser by navigating to the external IP address.

---

## **Terraform Code Explanation**

1. **VPC and Subnets**: 
    - The `main.tf` file contains the Terraform code for provisioning a new VPC, subnets, and related network infrastructure that is necessary for EKS.

2. **EKS Cluster**:
    - The EKS cluster is created using the AWS EKS module. It provisions an EKS control plane with node groups.

3. **IAM Roles**:
    - IAM roles and policies are defined to allow the EKS worker nodes to interact with other AWS resources like S3 and ECR.

4. **Outputs**:
    - After applying the Terraform configuration, you will get the EKS cluster details in the output, which can be used for `kubectl` configuration.

---

## **Jenkins Pipeline Explanation**

The Jenkins pipeline is responsible for automating the build, push, and deployment processes.

1. **Stage 1: Clone Repository**:
    - The first step in the pipeline is to clone the GitHub repository containing the application source code, Dockerfile, and Kubernetes manifest files.

    ```groovy
    git branch: 'main', url: 'https://github.com/sammikenzie/terraform-eks.git'
    ```

2. **Stage 2: Build Docker Image**:
    - In this stage, Jenkins builds the Docker image for the Flask application and tags it with `latest`.

    ```groovy
    dockerImage = docker.build("${env.ECR_REPO_NAME}:${env.IMAGE_TAG}")
    ```

3. **Stage 3: Login to Amazon ECR**:
    - Jenkins logs into Amazon ECR using AWS credentials stored on the Jenkins server. This allows the pipeline to push the Docker image to ECR.

    ```groovy
    sh "aws ecr get-login-password --region ${env.AWS_REGION} | docker login --username AWS --password-stdin ${env.AWS_ACCOUNT_ID}.dkr.ecr.${env.AWS_REGION}.amazonaws.com"
    ```

4. **Stage 4: Push Docker Image to ECR**:
    - The Docker image is tagged and pushed to the Amazon ECR repository.

    ```groovy
    sh "docker tag ${env.ECR_REPO_NAME}:${env.IMAGE_TAG} ${env.AWS_ACCOUNT_ID}.dkr.ecr.${env.AWS_REGION}.amazonaws.com/${env.ECR_REPO_NAME}:${env.IMAGE_TAG}"
    sh "docker push ${env.AWS_ACCOUNT_ID}.dkr.ecr.${env.AWS_REGION}.amazonaws.com/${env.ECR_REPO_NAME}:${env.IMAGE_TAG}"
    ```

5. **Stage 5: Deploy to Kubernetes**:
    - After the Docker image is pushed to ECR, Jenkins updates the running application in the EKS cluster by updating the image of the Kubernetes deployment.

    ```groovy
    sh "kubectl set image deployment/hello-world-deployment hello-world=${env.AWS_ACCOUNT_ID}.dkr.ecr.${env.AWS_REGION}.amazonaws.com/${env.ECR_REPO_NAME}:${env.IMAGE_TAG} --kubeconfig ${env.KUBECONFIG_PATH}"
    ```

6. **Stage 6: Clean Workspace**:
    - Jenkins cleans up the workspace after each build to ensure no unnecessary files are retained between builds.

    ```groovy
    cleanWs()
    ```

---

## **Conclusion**

This project demonstrates how to use Docker, Jenkins, Terraform, and AWS EKS to build and deploy a simple Flask web application. The CI/CD pipeline ensures that new changes are automatically built, tested, and deployed to production.

Feel free to reach out if you encounter any issues while setting up the environment or deploying the application.

---

Let me know if you'd like to modify any part of the README!
