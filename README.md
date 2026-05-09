# Blue-Green Deployment Demo using Kubernetes

## What the project is
Designed and implemented a Blue-Green Deployment architecture to achieve zero-downtime application releases using DevOps and cloud-native practices. The project simulated real-world production deployment strategies by maintaining two identical environments (Blue and Green) and shifting traffic between them during application updates.

## Step instructions
Built and containerized the application using Docker, automated CI/CD pipelines with Jenkins, and orchestrated deployments using Kubernetes. Configured load balancing and traffic switching using Kubernetes Services/Ingress to ensure seamless user experience during deployments and instant rollback capability in case of failures

## What did the project demonstrate
The project demonstrated practical understanding of deployment automation, rollback strategies, high availability, container orchestration, and modern DevOps workflows.

## Architecture

[![Architecture Diagram](/docs/images/blue-green-deployment.png)](/docs/images/blue-green-deployment.png)

### Tech Stack
Docker, Kubernetes, Jenkins, ALB Ingress, AWS EKS, Git, Linux

### Key Highlights

1. Implemented zero-downtime deployments
2. Automated CI/CD pipeline for application delivery
3. Configured rollback mechanism for failed releases
4. Demonstrated traffic switching between Blue and Green environments

## Deployment Workflow
1. Current version runs in Blue environment
2. New version deployed to Green
3. Health checks executed
4. Traffic switched to Green
5. Rollback possible if failures occur