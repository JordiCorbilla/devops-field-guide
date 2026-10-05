# Terraform Cheat Sheet

## Core workflow

```bash
terraform version
terraform init
terraform fmt -recursive
terraform validate
terraform plan
terraform apply
```

A safer review/apply flow:

```bash
terraform plan -out=tfplan
terraform show tfplan
terraform apply tfplan
```

## Initialize / upgrade

```bash
terraform init
terraform init -upgrade
terraform init -reconfigure
```

Validate a module without contacting its configured backend:

```bash
terraform init -backend=false
terraform validate
```

## Format / validate

```bash
terraform fmt
terraform fmt -recursive
terraform fmt -check -recursive
terraform validate
```

## Plan

```bash
terraform plan
terraform plan -out=tfplan
terraform show tfplan
terraform show -json tfplan | jq .
```

Plan with variables:

```bash
terraform plan -var='environment=dev'
terraform plan -var-file=dev.tfvars
```

Refresh-only:

```bash
terraform plan -refresh-only
terraform apply -refresh-only
```

Replace a specific resource:

```bash
terraform plan -replace='aws_instance.example'
terraform apply -replace='aws_instance.example'
```

Prefer `-replace` over old `terraform taint` workflows.

## Apply

```bash
terraform apply
terraform apply tfplan
```

Avoid `-auto-approve` interactively unless the execution context is tightly controlled.

## Destroy

Preview:

```bash
terraform plan -destroy
```

**DANGER:**

```bash
terraform destroy
```

## Workspaces

```bash
terraform workspace list
terraform workspace show
terraform workspace new <name>
terraform workspace select <name>
```

Do not assume workspaces are the isolation mechanism in every codebase; many organizations use separate backends/directories/accounts instead.

## Outputs

```bash
terraform output
terraform output <name>
terraform output -json
```

Sensitive outputs can still be obtained by authorized users; `sensitive = true` mainly suppresses routine display.

## State inspection

```bash
terraform state list
terraform state show <address>
```

Move:

```bash
terraform state mv <source> <destination>
```

Remove from state without deleting the real resource:

```bash
terraform state rm <address>
```

**DANGER:** `state mv` and `state rm` directly alter Terraform's state relationship. Back up and understand the backend/locking model first.

Pull state:

```bash
terraform state pull > state-backup.json
```

Treat state as sensitive: it can contain credentials or secrets depending on providers/resources.

## Import

CLI import:

```bash
terraform import <resource-address> <remote-id>
```

Modern import block:

```hcl
import {
  to = aws_s3_bucket.example
  id = "my-bucket"
}
```

Then:

```bash
terraform plan
```

## Providers

```bash
terraform providers
terraform providers lock
```

## Console

```bash
terraform console
```

Useful for testing expressions:

```hcl
> cidrsubnet("10.0.0.0/16", 8, 3)
```

## Dependency graph

```bash
terraform graph > graph.dot
dot -Tpng graph.dot -o graph.png
```

## Targeting

```bash
terraform plan -target=<resource-address>
terraform apply -target=<resource-address>
```

Use `-target` as an exceptional recovery/tooling mechanism, not normal deployment practice. It can create partially reconciled infrastructure.

## Lock recovery

If a lock is genuinely stale:

```bash
terraform force-unlock <LOCK_ID>
```

**DANGER:** never force-unlock simply because another valid run is taking a long time.

## Useful CI sequence

```bash
terraform fmt -check -recursive
terraform init -backend=false
terraform validate
```

For a real environment plan, initialize the real backend under the pipeline's controlled credentials, then plan.
