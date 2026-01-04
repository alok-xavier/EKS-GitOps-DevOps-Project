# Cloud-Native GitOps Deployment on AWS EKS

This project demonstrates an end-to-end DevOps workflow for deploying
containerized applications on AWS EKS using Terraform, CI pipelines,
and GitOps-based continuous delivery with ArgoCD.

---

## Repository Structure

eks terraform install/
└── eks-install/
    └── Terraform code for provisioning AWS EKS cluster

project/
├── .github/workflows
│   └── ci.yaml              # CI pipeline
├── kubernetes manifests     # Kubernetes deployment manifests
├── src                      # Sample application source
└── internal                 # Supporting modules

---

## Workflow Overview

1. Terraform provisions AWS EKS cluster
2. CI pipeline builds Docker images
3. Kubernetes manifests are version-controlled in Git
4. ArgoCD syncs and deploys changes to EKS

---

## Technologies Used

- AWS EKS
- Terraform
- Kubernetes
- Docker
- GitHub Actions (CI)
- ArgoCD (GitOps CD)

---

## Future Enhancements

- Canary deployments using Istio
- Observability and monitoring integration
