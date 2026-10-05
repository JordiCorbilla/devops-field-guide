# First-Day Checklist

The goal is not to prove you remember commands. The goal is to avoid operating on the wrong account, cluster, namespace, workspace, branch, or container.

## 1. Identity and credentials

### AWS

```bash
aws sts get-caller-identity
aws configure list
aws configure list-profiles
```

For SSO:

```bash
aws sso login --profile <profile>
aws sts get-caller-identity --profile <profile>
```

### Git

```bash
git config --get user.name
git config --get user.email
git remote -v
```

## 2. Kubernetes context

```bash
kubectl config get-contexts
kubectl config current-context
kubectl cluster-info
kubectl get ns
```

Check the active namespace:

```bash
kubectl config view --minify --output 'jsonpath={..namespace}'; echo
```

Use an explicit namespace when in doubt:

```bash
kubectl get pods -n <namespace>
```

## 3. Docker context

```bash
docker context ls
docker context show
docker info
docker ps
```

## 4. Terraform state/workspace

```bash
terraform version
terraform workspace show
terraform workspace list
terraform state list
```

Do **not** run `apply`, `destroy`, `state rm`, `state mv`, or `force-unlock` until you know where state lives and who else may be using it.

## 5. Repository state

```bash
git status
git branch --show-current
git log --oneline --decorate -10
git fetch --all --prune
```

## 6. Useful discovery commands

```bash
# Kubernetes
kubectl get all -n <namespace>
kubectl get events -n <namespace> --sort-by=.metadata.creationTimestamp

# AWS
aws eks list-clusters --region <region>
aws ec2 describe-regions --query 'Regions[].RegionName' --output text

# Docker
docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Ports}}"
docker system df

# Terraform
terraform providers
terraform output
```

## 7. Before any production change

Confirm:

1. Account / tenant.
2. Region.
3. Kubernetes context.
4. Namespace.
5. Terraform workspace/backend.
6. Git branch/commit.
7. Exact resource.
8. Rollback route.
9. Observability/logs ready.
10. Another engineer knows if the change is risky.
