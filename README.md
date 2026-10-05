# DevOps Field Guide

A compact operational cheat sheet for Docker, Kubernetes, AWS, Terraform, PowerShell, Git, and Linux/network troubleshooting.

This is deliberately command-first: the sort of repository you can keep open during your first week in a new environment.

> **Rule zero:** before changing anything, verify **identity**, **environment/context**, **namespace/workspace**, and **target resource**.

## Contents

- [First-day / before-you-touch-anything checklist](docs/first-day-checklist.md)
- [Docker](docs/docker.md)
- [Kubernetes](docs/kubernetes.md)
- [AWS CLI / EKS / SSM / CloudWatch](docs/aws.md)
- [Terraform](docs/terraform.md)
- [PowerShell](docs/powershell.md)
- [Git](docs/git.md)
- [Linux & networking](docs/linux-networking.md)
- [Troubleshooting playbook](docs/troubleshooting.md)

## The 30-second environment check

```bash
# Git
git status
git branch --show-current
git remote -v

# Docker
docker context show
docker ps

# AWS
aws sts get-caller-identity
aws configure list

# Kubernetes
kubectl config current-context
kubectl config view --minify --output 'jsonpath={..namespace}'; echo
kubectl get pods

# Terraform
terraform workspace show
terraform version
```

## Conventions

- `<>` means replace with your value.
- Commands marked **DANGER** can delete, recreate, overwrite, or expose resources/data.
- Prefer explicit `--profile`, `--region`, Kubernetes `--context`, and `-n <namespace>` in unfamiliar environments.
- Avoid pasting secrets into terminals with persistent history.

## Official references

- Docker CLI: https://docs.docker.com/reference/cli/docker/
- Kubernetes kubectl: https://kubernetes.io/docs/reference/kubectl/
- AWS CLI: https://docs.aws.amazon.com/cli/latest/reference/
- Terraform CLI: https://developer.hashicorp.com/terraform/cli/commands
- PowerShell: https://learn.microsoft.com/powershell/
