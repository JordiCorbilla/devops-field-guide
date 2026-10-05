# AWS CLI / EKS Cheat Sheet

## Identity first

```bash
aws sts get-caller-identity
aws configure list
aws configure list-profiles
```

With explicit profile/region:

```bash
aws sts get-caller-identity --profile <profile>
aws configure get region --profile <profile>
```

SSO:

```bash
aws sso login --profile <profile>
```

Useful environment variables:

```bash
export AWS_PROFILE=<profile>
export AWS_REGION=eu-west-2
export AWS_PAGER=""
```

PowerShell:

```powershell
$env:AWS_PROFILE = "<profile>"
$env:AWS_REGION = "eu-west-2"
$env:AWS_PAGER = ""
```

## EKS

List clusters:

```bash
aws eks list-clusters --region <region>
```

Describe:

```bash
aws eks describe-cluster --name <cluster> --region <region>
```

Create/update kubeconfig:

```bash
aws eks update-kubeconfig --name <cluster> --region <region>
```

With role:

```bash
aws eks update-kubeconfig \
  --name <cluster> \
  --region <region> \
  --role-arn arn:aws:iam::<account-id>:role/<role>
```

Then verify:

```bash
kubectl config current-context
kubectl get nodes
```

## ECR

Authenticate Docker:

```bash
aws ecr get-login-password --region <region> \
  | docker login --username AWS --password-stdin <account>.dkr.ecr.<region>.amazonaws.com
```

Repositories:

```bash
aws ecr describe-repositories --region <region>
aws ecr list-images --repository-name <repo> --region <region>
```

## CloudWatch Logs

Tail a log group:

```bash
aws logs tail <log-group> --follow
aws logs tail <log-group> --since 10m
aws logs tail <log-group> --since 1h --follow
```

Filter:

```bash
aws logs filter-log-events \
  --log-group-name <log-group> \
  --filter-pattern "ERROR"
```

List groups:

```bash
aws logs describe-log-groups --log-group-name-prefix <prefix>
```

## S3

```bash
aws s3 ls
aws s3 ls s3://<bucket>/<prefix>/
aws s3 cp file.txt s3://<bucket>/<prefix>/
aws s3 cp s3://<bucket>/<key> .
```

Recursive copy:

```bash
aws s3 cp ./dir s3://<bucket>/<prefix>/ --recursive
```

Sync:

```bash
aws s3 sync ./dir s3://<bucket>/<prefix>/
aws s3 sync s3://<bucket>/<prefix>/ ./dir
```

Preview deletes before doing anything destructive:

```bash
aws s3 sync ./dir s3://<bucket>/<prefix>/ --delete --dryrun
```

**DANGER:**

```bash
aws s3 sync ./dir s3://<bucket>/<prefix>/ --delete
aws s3 rm s3://<bucket>/<prefix>/ --recursive
```

## EC2

Instances:

```bash
aws ec2 describe-instances \
  --query 'Reservations[].Instances[].{Id:InstanceId,State:State.Name,Type:InstanceType,PrivateIp:PrivateIpAddress,Name:Tags[?Key==`Name`]|[0].Value}' \
  --output table
```

Security groups:

```bash
aws ec2 describe-security-groups --group-ids <sg-id>
```

## Systems Manager Session Manager

Start a shell without opening SSH:

```bash
aws ssm start-session --target <instance-id>
```

Port forwarding:

```bash
aws ssm start-session \
  --target <instance-id> \
  --document-name AWS-StartPortForwardingSession \
  --parameters '{"portNumber":["5432"],"localPortNumber":["15432"]}'
```

## Secrets Manager

Metadata only:

```bash
aws secretsmanager list-secrets
aws secretsmanager describe-secret --secret-id <secret-id>
```

Retrieve a secret only when operationally necessary:

```bash
aws secretsmanager get-secret-value --secret-id <secret-id>
```

Be aware this can print credentials into terminal scrollback/history/log capture.

## Parameter Store

```bash
aws ssm get-parameter --name <name>
aws ssm get-parameter --name <name> --with-decryption
aws ssm get-parameters-by-path --path <path> --recursive
```

## ECS

Clusters/services:

```bash
aws ecs list-clusters
aws ecs list-services --cluster <cluster>
aws ecs describe-services --cluster <cluster> --services <service>
```

Tasks:

```bash
aws ecs list-tasks --cluster <cluster> --service-name <service>
aws ecs describe-tasks --cluster <cluster> --tasks <task-arn>
```

Exec into a task if ECS Exec is enabled:

```bash
aws ecs execute-command \
  --cluster <cluster> \
  --task <task-id> \
  --container <container> \
  --interactive \
  --command "/bin/sh"
```

## CloudFormation

```bash
aws cloudformation list-stacks \
  --stack-status-filter CREATE_COMPLETE UPDATE_COMPLETE

aws cloudformation describe-stacks --stack-name <stack>
aws cloudformation describe-stack-events --stack-name <stack>
```

## Handy querying

JMESPath:

```bash
aws ec2 describe-instances \
  --query 'Reservations[].Instances[?State.Name==`running`].[InstanceId,PrivateIpAddress]' \
  --output table
```

JSON + jq:

```bash
aws eks describe-cluster --name <cluster> --output json | jq '.cluster | {name,status,version,endpoint}'
```
