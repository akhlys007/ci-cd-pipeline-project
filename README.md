# 🚀 AWS CI/CD Pipeline for a Node.js App using ECS Fargate, ECR, and CodePipeline

This project demonstrates how to deploy a simple Node.js application using Docker containers on AWS ECS (with Fargate), managed via a complete CI/CD pipeline using **CodeCommit**, **CodeBuild**, **CodePipeline**, and **ECR**. It automates container builds and deployments on code push.

[Live Demo of the completed Node.js App](http://nodejs-server-alb-1-1198286681.eu-north-1.elb.amazonaws.com)
## 📌 Table of Contents
- [Architecture Overview](#architecture-overview)
- [Project Structure](#project-structure)
- [Prerequisites](#prerequisites)
- [Step-by-Step Setup](#step-by-step-setup)
  - [1. Clone the Project](#1-clone-the-project)
  - [2. Dockerize the Application](#2-dockerize-the-application)
  - [3. Push to Amazon ECR](#3-push-to-amazon-ecr)
  - [4. Set Up ECS and Fargate](#4-set-up-ecs-and-fargate)
  - [5. Create CI/CD Pipeline](#5-create-cicd-pipeline)
- [File References](#file-references)
- [Screenshots / Diagrams](#screenshots--diagrams)
- [License](#license)

## 📐 Architecture Overview

This is a basic representation of the architecture used in this project:

```
GitHub (Source) 
   │
   ▼
AWS CodePipeline ──▶ CodeBuild ──▶ ECR ──▶ ECS (Fargate) + ALB
                                ▲
                                │
                            Docker Image
```

> 🖼️ **[Insert architecture diagram here]**

## 🗂️ Project Structure

```
├── Dockerfile
├── buildspec.yml
├── app.js
├── package.json
```

The Node.js app exposes a simple HTTP server on port `3000`.

## ✅ Prerequisites

Make sure you have the following:

- AWS account
- AWS CLI configured (`aws configure`)
- IAM user with permissions for:
  - ECR
  - ECS
  - CodeCommit/CodeBuild/CodePipeline
- Docker installed
- Git installed
- An existing GitHub repository:  
  👉 [https://github.com/akhlys007/ci-cd-pipeline-project](https://github.com/akhlys007/ci-cd-pipeline-project)

## 🛠️ Step-by-Step Setup

### 1. Clone the Project

```bash
git clone https://github.com/akhlys007/ci-cd-pipeline-project.git
cd ci-cd-pipeline-project
```

### 2. Dockerize the Application

```Dockerfile
FROM node:alpine

WORKDIR /nodejs-docker-aws-ecs

COPY package.json .

RUN npm install

COPY . .

EXPOSE 3000

CMD [ "node", "app.js" ]
```

Build and run locally to test:

```bash
docker build -t nodejs-server-demo .
docker run -p 3000:3000 nodejs-server-demo
```

### 3. Push to Amazon ECR

#### A. Create an ECR repository

```bash
aws ecr create-repository --repository-name nodejs-server-demo-private
```

#### B. Authenticate Docker to ECR

```bash
aws ecr get-login-password --region eu-north-1 | docker login --username AWS --password-stdin 474668403494.dkr.ecr.eu-north-1.amazonaws.com
```

#### C. Tag and Push

```bash
docker tag nodejs-server-demo:latest 474668403494.dkr.ecr.eu-north-1.amazonaws.com/nodejs-server-demo-private:latest
docker push 474668403494.dkr.ecr.eu-north-1.amazonaws.com/nodejs-server-demo-private:latest
```

### 4. Set Up ECS and Fargate

- Create an ECS Cluster (Fargate)
- Create a Task Definition using the image above
- Expose port `3000`
- Create a Service with 2 replicas
- Use Application Load Balancer (ALB)
- Configure security groups appropriately

> 📸 **[Insert ECS screenshot here]**

### 5. Create CI/CD Pipeline

#### A. CodePipeline Stages:
1. Source: GitHub
2. Build: CodeBuild
3. Deploy: ECS

#### B. Add `buildspec.yml`

```yaml
version: 0.2

phases:
  pre_build:
    commands:
      - echo Logging in to Amazon ECR...
      - aws --version
      - aws ecr get-login-password --region $AWS_DEFAULT_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com
      - REPOSITORY_URI=$AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME
  build:
    commands:
      - echo Build started on `date`
      - echo Building the Docker image...
      - docker build -t $REPOSITORY_URI:$IMAGE_TAG .
      - docker tag $REPOSITORY_URI:$IMAGE_TAG $REPOSITORY_URI:$IMAGE_TAG
  post_build:
    commands:
      - echo Build completed on `date`
      - echo Pushing the Docker images...
      - docker push $REPOSITORY_URI:$IMAGE_TAG
      - echo Writing image definitions file...
      - printf '[{"name":"%s","imageUri":"%s"}]' $CONTAINER_NAME $REPOSITORY_URI:$IMAGE_TAG > imagedefinitions.json

artifacts:
  files: imagedefinitions.json
```

#### C. CodeBuild Environment Variables

| Key                | Value                                                                 |
|--------------------|------------------------------------------------------------------------|
| `AWS_ACCOUNT_ID`   | `private`                                                         |
| `AWS_DEFAULT_REGION` | `eu-north-1`                                                        |
| `IMAGE_REPO_NAME`  | `nodejs-server-demo-private`                                           |
| `REPOSITORY_URI`   | `private.dkr.ecr.eu-north-1.amazonaws.com/nodejs-server-demo-private` |
| `IMAGE_TAG`        | `latest`                                                               |
| `CONTAINER_NAME`   | `nodejs-server-demo-private`                                           |

## 📂 File References

- `Dockerfile`: Defines the container build process
- `buildspec.yml`: Defines build & push steps for CodeBuild
- `app.js`: Node.js app entry point
- `package.json`: Project dependencies

## 🖼️ Screenshots / Diagrams

> 📸 **[Placeholder: ECS Cluster Screenshot]**  
> 📸 **[Placeholder: CodePipeline Screenshot]**  
> 🗺️ **[Placeholder: Architecture Diagram]**

## 📄 License

