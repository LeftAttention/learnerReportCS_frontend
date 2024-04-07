# LRC Frontend Deployment Documentation

## Overview

This document outlines the process for deploying a React.js frontend application on AWS Elastic Kubernetes Service (EKS) using Helm. The deployment includes setting up a LoadBalancer service to expose the application externally and configuring it to communicate dynamically with the backend service.

## Prerequisites

- AWS CLI installed and configured
- kubectl configured to interact with the Kubernetes cluster
- Helm installed
- Docker image of the frontend application pushed to Amazon Elastic Container Registry (ECR)

## Steps

### 1. Create an EKS Cluster

This step remains the same as the backend deployment.

### 2. Prepare the Docker Image

Replace `lrc-backend` with `lrc-frontend` for all the Docker-related commands.

2.1. **Build the Docker Image:**
```bash
docker build -t lrc-frontend:latest .
```

2.2. **Tag the Image:**
```bash
docker tag lrc-frontend:latest AWS_ACCOUNT_ID.dkr.ecr.REGION.amazonaws.com/lrc-frontend:latest
```

2.3. **Push the Image to ECR:**
```bash
docker push AWS_ACCOUNT_ID.dkr.ecr.REGION.amazonaws.com/lrc-frontend:latest
```

### 3. Create a Helm Chart for the Frontend Application

Replace `lrc-backend` with `lrc-frontend` in Helm-related commands.

3.1. **Initialize Helm Chart:**
```bash
helm create lrc-frontend
```

3.2. **Edit `values.yaml` to set the image and service configuration:**

```yaml
replicaCount: 2

image:
  repository: AWS_ACCOUNT_ID.dkr.ecr.REGION.amazonaws.com/lrc-frontend
  pullPolicy: IfNotPresent
  tag: latest

service:
  type: LoadBalancer
  port: 80
  targetPort: 80
```

### 4. Deploy the Frontend Application using Helm

4.1. **Install the Helm Chart:**
```bash
helm install lrc-frontend ./lrc-frontend
```

4.2. **Verify the Deployment:**
```bash
kubectl get deployments
```

4.3. **Check the Service and Load Balancer:**
```bash
kubectl get svc
```

### 5. Configure Frontend to Communicate with Backend

5.1. **Set the Backend URL Dynamically:**
To dynamically configure the frontend to communicate with the backend, update the `config.env` file in the frontend application with the following command:

```bash
export SERVICE_IP=$(kubectl get svc --namespace default lrc-backend --template "{{ range (index .status.loadBalancer.ingress 0) }}{{.}}{{ end }}")
echo "BACKEND_URL=http://$SERVICE_IP" >> config.env
```

### 6. Access the Application

6.1. **Retrieve the Load Balancer URL:**
```bash
kubectl get service lrc-frontend -o wide
```

The external IP address or hostname listed under `EXTERNAL-IP` is the URL through which the frontend application can be accessed.

### 7. Monitoring and Management

This section remains the same as the backend deployment, replacing `lrc-backend` with `lrc-frontend` where necessary.

## Troubleshooting

The troubleshooting steps are similar to the backend, with the necessary adjustments for the frontend.

## Conclusion

This document covers the deployment of a React.js frontend application on AWS EKS using Helm, including setting up a LoadBalancer to expose the application and configuring it to dynamically communicate with the backend service. For detailed monitoring, scaling, or updating the frontend application, refer to Kubernetes and AWS documentation.