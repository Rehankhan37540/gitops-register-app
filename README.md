
# GitOps Register App — Kubernetes Deployment on Amazon EKS

This repository manages the Kubernetes deployment configuration for the Register App project. It demonstrates a GitOps-based Continuous Deployment (CD) workflow using Jenkins, GitHub, Argo CD, Kubernetes, and Amazon EKS.

The repository acts as the source of truth for the application's Kubernetes Deployment and Service configuration.

## Overview

The application CI pipeline builds and publishes a Docker image. A separate Jenkins CD job receives the new image tag, updates the Kubernetes Deployment manifest, and pushes the change to this repository.

Argo CD monitors the GitOps repository and synchronizes the desired configuration with the Amazon EKS cluster.

This separates application build activities from Kubernetes deployment configuration.

## Technology Stack

| Technology | Purpose |
|---|---|
| Git | Version control for deployment configuration |
| GitHub | GitOps configuration repository |
| Jenkins | Automates Deployment manifest updates |
| Docker Hub | Stores versioned application images |
| Kubernetes | Runs and manages application containers |
| Amazon EKS | Managed Kubernetes cluster |
| Argo CD | Reconciles Git configuration with cluster state |
| AWS Load Balancer | Exposes the application externally |

## GitOps Workflow

1. **CI Image Published:** The application CI pipeline builds and publishes a Docker image to Docker Hub.
2. **CD Job Triggered:** Jenkins triggers the separate `gitops-register-app-cd` job and passes the `IMAGE_TAG` parameter.
3. **Repository Checkout:** The CD job checks out the GitOps repository's `main` branch.
4. **Image Version Updated:** Jenkins updates the image reference in `deployment.yaml` using the supplied tag.
5. **Git Commit and Push:** The changed Deployment manifest is committed and pushed to GitHub.
6. **Argo CD Synchronization:** Argo CD detects the repository change and synchronizes the desired Kubernetes configuration.
7. **Kubernetes Rollout:** Kubernetes updates the running application according to the Deployment configuration.
8. **Application Access:** The Kubernetes Service exposes the application through a LoadBalancer.

## Repository Structure

```text
gitops-register-app/
├── Jenkinsfile
├── deployment.yaml
├── service.yaml
└── README.md
```

### File Descriptions

| File | Responsibility |
|---|---|
| `deployment.yaml` | Defines the application Deployment, replica count, image, container port, and resource limits. |
| `service.yaml` | Defines the Kubernetes Service that exposes the application through a LoadBalancer. |
| `Jenkinsfile` | Automates the image tag update and pushes the modified Deployment manifest to GitHub. |

## Kubernetes Configuration

### Deployment

The current `deployment.yaml` defines:

- Deployment name: `register-app-deployment`
- Desired replicas: `2`
- Container image repository: `rehankhan37237/register-app-pipeline`
- Container port: `8080`
- Memory limit: `256Mi`
- CPU limit: `500m`

The image tag is updated by the CD pipeline. The exact tag in the repository can change with subsequent deployments.

### Service

The current `service.yaml` defines:

- Service name: `register-app-service`
- Service type: `LoadBalancer`
- Service port: `8080`
- Target port: `8080`

The Service selects the application pods using the `app: register-app` label.

## Image Version Management

The application CI pipeline publishes versioned images using the pattern:

`1.0.0-<build-number>`

For example:

`rehankhan37237/register-app-pipeline:1.0.0-121`

The CD job updates the Deployment manifest with the supplied version. This keeps the image reference in Git aligned with the intended application version.

Versioned tags also help identify earlier releases when a rollback is needed.

## Argo CD Integration

Argo CD is used to monitor the GitOps repository and synchronize the desired application configuration to Amazon EKS.

For this workflow, the Argo CD Application must be configured to track the GitOps repository and the directory containing the Kubernetes manifests.

The repository contains the Deployment and Service manifests at its root. The Argo CD Application configuration and cluster access are managed in the deployment environment.

## Prerequisites

To reproduce the workflow, the following components and access are required:

- A GitHub account with access to the application and GitOps repositories.
- A Jenkins controller and an available build agent.
- Jenkins credentials configured for GitHub repository access.
- Docker Hub access to the published application image.
- An Amazon EKS cluster with working Kubernetes access.
- Argo CD configured to monitor this GitOps repository.
- Appropriate AWS and Kubernetes permissions.

Infrastructure provisioning and Jenkins/Argo CD credential setup are managed outside this repository.

## Deployment Flow at a Glance

```text
Application Source
       |
       v
Jenkins CI Pipeline
       |
       v
Docker Image Published
       |
       v
Jenkins GitOps CD Job
       |
       v
Update deployment.yaml
       |
       v
Commit and Push to GitHub
       |
       v
Argo CD Detects Change
       |
       v
Amazon EKS Synchronizes
       |
       v
Kubernetes Application Rollout
```

## Rollback Approach

Because the Deployment manifest references a versioned image, a previous release can be restored by updating the image reference to a previously published image tag and committing that change to the GitOps repository.

Argo CD can then synchronize the restored desired configuration with the cluster, provided the previous image is still available and the application configuration remains compatible.

## Security Best Practices

- Store GitHub credentials and Jenkins API tokens in Jenkins Credentials.
- Do not commit passwords, tokens, private keys, or AWS access keys.
- Use least-privilege IAM and Kubernetes permissions.
- Review image vulnerability scan results before promoting an image.
- Keep deployment changes in Git so that configuration updates are traceable.

## Related Repository

**Application source and CI pipeline:**  
[https://github.com/Rehankhan37540/register-app](https://github.com/Rehankhan37540/register-app)

## Key Learning Outcomes

- Managing Kubernetes manifests through Git.
- Automating image tag updates with Jenkins.
- Separating CI from CD responsibilities.
- Using Argo CD to synchronize desired state.
- Deploying containerized applications to Amazon EKS.
- Using versioned images to support traceability and rollback.
- Troubleshooting Kubernetes deployments and external Service access.

## Author

**Rehan Khan**

Aspiring Cloud Engineer | DevOps | AWS | Kubernetes | CI/CD | GitOps

---

This repository is part of my hands-on learning journey in GitOps, Kubernetes automation, and cloud-native application delivery.
