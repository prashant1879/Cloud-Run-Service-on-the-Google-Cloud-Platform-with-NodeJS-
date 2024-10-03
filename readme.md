# **Deploying a Cloud Run Service on Google Cloud Platform**

This repository contains a sample project demonstrating how to deploy a Node.js application as a containerized service on Google Cloud Platform (GCP) using **Cloud Run** and **Google Artifact Registry**.

## **Overview**
Cloud Run is a fully managed compute platform that allows you to run containerized applications in a serverless environment. This project provides a simple Node.js application that responds to HTTP requests and shows how to:

1. Containerize the application using Docker.
2. Push the Docker image to Google Artifact Registry (GCP’s container registry).
3. Deploy the containerized application to Cloud Run.

## **Pre-requisites**

Before deploying, ensure you have the following:

- **Google Cloud SDK** installed and authenticated.
- A **GCP project** set up.
- **Docker** installed on your machine.
- **Artifact Registry API** enabled in your GCP project.

## **Steps to Deploy**

### **1. Set Up Your Environment**
- Authenticate your Google Cloud SDK with your GCP account:
  ```bash
  gcloud auth login
  ```
- Set your project:
  ```bash
  gcloud config set project [PROJECT_ID]
  ```

### **2. Write and Containerize Your Application**
- Create a simple Node.js application. The project includes a file named `app.js` that responds to HTTP GET requests.
  
  Example `app.js`:
  ```javascript
  const express = require('express');
  const app = express();
  
  app.get('/', (req, res) => {
    res.send('Hello from Cloud Run!');
  });

  const port = process.env.PORT || 8080;
  app.listen(port, () => {
    console.log(`Server is running on port ${port}`);
  });
  ```

- Add a `Dockerfile` to containerize the application:
  ```dockerfile
  FROM node:18
  WORKDIR /usr/src/app
  COPY package*.json ./
  RUN npm install --production
  COPY . .
  ENV PORT 8080
  CMD ["node", "app.js"]
  ```

### **3. Set Up Google Artifact Registry**
- Enable the Artifact Registry API:
  ```bash
  gcloud services enable artifactregistry.googleapis.com
  ```
- Create a Docker repository:
  ```bash
  gcloud artifacts repositories create [REPO_NAME] \
  --repository-format=docker \
  --location=[REGION] \
  --description="Docker repository"
  ```

### **4. Build and Push Your Docker Image**
- Build the Docker image:
  ```bash
  docker build -t [REGION]-docker.pkg.dev/[PROJECT_ID]/[REPO_NAME]/[IMAGE_NAME] .
  ```
- Push the image to Artifact Registry:
  ```bash
  docker push [REGION]-docker.pkg.dev/[PROJECT_ID]/[REPO_NAME]/[IMAGE_NAME]
  ```

### **5. Deploy to Cloud Run**
- Deploy the containerized image to Cloud Run:
  ```bash
  gcloud run deploy [SERVICE_NAME] \
  --image [REGION]-docker.pkg.dev/[PROJECT_ID]/[REPO_NAME]/[IMAGE_NAME] \
  --platform managed \
  --region [REGION] \
  --allow-unauthenticated
  ```

### **6. Access Your Cloud Run Service**
- After deployment, Cloud Run will output a URL for your service. Visit the URL in your browser to see your deployed application.

## **Project Structure**

```
├── app.js                # Node.js application code
├── Dockerfile            # Dockerfile to containerize the app
├── package.json          # Node.js dependencies and scripts
└── README.md             # This file
```

## **Common Commands**

- **Update the Docker Image**:
  ```bash
  docker build -t [REGION]-docker.pkg.dev/[PROJECT_ID]/[REPO_NAME]/[IMAGE_NAME] .
  docker push [REGION]-docker.pkg.dev/[PROJECT_ID]/[REPO_NAME]/[IMAGE_NAME]
  ```
- **Redeploy the Cloud Run Service**:
  ```bash
  gcloud run deploy [SERVICE_NAME] \
  --image [REGION]-docker.pkg.dev/[PROJECT_ID]/[REPO_NAME]/[IMAGE_NAME]
  ```

## **Logs and Monitoring**

- To monitor the logs of your service:
  ```bash
  gcloud logging read "resource.type=cloud_run_revision AND resource.labels.service_name=[SERVICE_NAME]"
  ```

## **Conclusion**

This project demonstrates how to deploy a containerized application to Cloud Run using Google Artifact Registry. With this setup, you can efficiently manage containerized applications with the benefits of serverless auto-scaling and minimal infrastructure management.

## **License**

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
