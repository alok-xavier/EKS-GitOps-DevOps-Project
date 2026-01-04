<!-- Copilot instructions for AI coding agents working on this repo -->
# Copilot Instructions — Terraform EKS repo

Purpose
- Help AI contributors be productive with the Terraform EKS code under `eks/`.

Quick architecture summary
- This repo provisions an EKS cluster using local Terraform modules in `eks/eks-install/modules`:
  - `eks-install/main.tf` composes two modules: `./modules/vpc` and `./modules/eks`.
  - `modules/vpc` creates a VPC, public/private subnets, NAT gateways and route tables.
  - `modules/eks` creates IAM roles, the `aws_eks_cluster` resource and `aws_eks_node_group` resources.
- State is managed with an S3 backend + DynamoDB locking configured in `eks/eks-install/main.tf` (bucket `demos-terraform-eks-state-s3-bucket`, table `terraform-eks-state-locks`).

Developer workflows (explicit)
- Work in the `eks/eks-install` directory for Terraform operations.
- Initialize and plan (example):

```bash
cd "eks/eks-install"
terraform init
terraform plan -out plan.tfplan
terraform apply "plan.tfplan"
```

- To access the cluster after apply, use the Terraform outputs and AWS CLI; example:

```bash
cd "eks/eks-install"
CLUSTER_NAME=$(terraform output -raw cluster_name)
aws eks update-kubeconfig --name "$CLUSTER_NAME" --region us-west-2
kubectl get nodes
```

Project-specific patterns & conventions
- Local-module layout: modules live under `eks/eks-install/modules/<name>` and are composed by `eks-install/main.tf`.
- Variable shapes are strict and important: `node_groups` is a map of objects with keys `instance_types`, `capacity_type`, and `scaling_config` (`desired_size`, `max_size`, `min_size`). Example shape in `eks-install/variables.tf`.
- Outputs are used to wire module values (e.g., `module.vpc.private_subnet_ids` passed to the EKS module). Prefer using module outputs rather than reconstructing IDs.
- Tagging convention: resources in `modules/vpc` add tags like `kubernetes.io/cluster/${var.cluster_name} = "shared"` — preserve these tags when modifying networking resources.

Integration points & external dependencies
- AWS provider (HashiCorp provider `hashicorp/aws` pinned in `eks/eks-install/main.tf`).
- Remote state S3 bucket and DynamoDB table names are configured in `eks/eks-install/main.tf`; avoid hard-changing these values without coordinating state migration.

What to avoid / safe-edit guidance
- Do not rename output or variable names lightly — other module references depend on them.
- When updating IAM policies or roles in `modules/eks`, ensure `depends_on` relationships remain correct to avoid race conditions on cluster creation.
- If changing the backend configuration (S3/DynamoDB), follow Terraform state migration best-practices — do not change the backend in-place without migrating state.

Where to look for examples
- `eks/eks-install/main.tf` — shows backend config and top-level composition.
- `eks/eks-install/modules/vpc/main.tf` — networking implementation and tagging patterns.
- `eks/eks-install/modules/eks/main.tf` — cluster, roles and node group examples including `for_each` usage for node groups.

If something is unclear
- Ask for the exact goal (e.g., "add a new node group called 'gpu' with spot instances"), and point to the specific module/file to change.

Feedback
- If any of the above examples or file paths are outdated, request which file to re-scan and I'll re-generate/update these instructions.
