# Troubleshooting Playbook

A fast, repeatable sequence is more valuable than memorizing isolated commands.

## General sequence

1. **Scope:** one user, one pod, one host, one AZ/region, or everything?
2. **Identity/context:** correct AWS account, cluster, namespace, Docker context, Terraform workspace?
3. **Recent change:** deploy, config, secret, image, dependency, DNS, certificate?
4. **Health/status:** what is failing now?
5. **Logs/events:** application + orchestrator/platform.
6. **Dependencies:** DNS, TCP, TLS, database, queue, API.
7. **Resources:** CPU, memory, disk, file descriptors, quotas.
8. **Rollback/restart:** only after collecting useful evidence.

---

# Docker container not working

```bash
docker ps -a
docker inspect <container>
docker logs --tail 200 <container>
docker top <container>
docker stats --no-stream <container>
```

Check exit code:

```bash
docker inspect -f '{{.State.ExitCode}}' <container>
```

Check health:

```bash
docker inspect -f '{{json .State.Health}}' <container>
```

Test shell:

```bash
docker exec -it <container> sh
```

If it exits too quickly, inspect image startup:

```bash
docker inspect -f '{{json .Config.Entrypoint}} {{json .Config.Cmd}}' <image>
docker run --rm -it --entrypoint sh <image>
```

---

# Kubernetes CrashLoopBackOff

```bash
kubectl get pod <pod> -o wide
kubectl describe pod <pod>
kubectl logs <pod> --previous
kubectl logs <pod> -c <container> --previous
kubectl get events --sort-by=.metadata.creationTimestamp
```

Common causes:

- bad startup command
- missing environment/config/secret
- dependency unavailable
- failed liveness probe
- OOMKilled
- permission/filesystem issue

Check last state:

```bash
kubectl get pod <pod> -o jsonpath='{.status.containerStatuses[*].lastState}'; echo
```

---

# Kubernetes pod is Pending

```bash
kubectl describe pod <pod>
kubectl get nodes
kubectl top nodes
kubectl get pvc
```

Look for:

- insufficient CPU/memory
- taints/tolerations
- node selectors/affinity
- unbound PVC
- topology constraints
- quotas

---

# Service is unreachable

Start from the caller.

DNS:

```bash
nslookup <host>
dig <host>
```

TCP:

```bash
nc -vz <host> <port>
```

HTTP/TLS:

```bash
curl -v https://<host>/health
openssl s_client -connect <host>:443 -servername <host>
```

In Kubernetes:

```bash
kubectl get svc <service> -o wide
kubectl get endpoints <service>
kubectl get endpointslice -l kubernetes.io/service-name=<service>
```

Then test from inside the cluster:

```bash
kubectl run tmp-shell --rm -it --restart=Never --image=busybox -- sh
```

---

# AWS authentication confusion

```bash
aws sts get-caller-identity
aws configure list
aws configure list-profiles
env | grep '^AWS_'
```

SSO:

```bash
aws sso login --profile <profile>
aws sts get-caller-identity --profile <profile>
```

EKS:

```bash
aws eks update-kubeconfig --name <cluster> --region <region> --profile <profile>
kubectl config current-context
kubectl auth can-i get pods -A
```

---

# Terraform says state is locked

Do not immediately force-unlock.

First determine:

- Is another CI job/apply running?
- Is a colleague applying?
- Did a previous job crash?
- What backend holds the lock?

Inspect pipeline/process history.

Only after confirming the lock is stale:

```bash
terraform force-unlock <LOCK_ID>
```

---

# Terraform plan is unexpectedly huge

Stop before apply.

Check:

```bash
terraform workspace show
terraform providers
terraform state list
terraform plan
```

Typical causes:

- wrong workspace/backend
- wrong account/region
- provider default changed
- resource address refactor without `moved` blocks/state move
- provider upgrade
- generated values/order changes
- lost/imported state

---

# High CPU/memory

Linux:

```bash
top
ps aux --sort=-%cpu | head
ps aux --sort=-%mem | head
free -h
```

Docker:

```bash
docker stats
docker top <container>
```

Kubernetes:

```bash
kubectl top nodes
kubectl top pods -A
kubectl top pod <pod> --containers
```

Check Kubernetes limits/requests:

```bash
kubectl get pod <pod> -o jsonpath='{range .spec.containers[*]}{.name}{" requests="}{.resources.requests}{" limits="}{.resources.limits}{"\n"}{end}'
```

---

# Disk full

```bash
df -h
df -i
du -sh /var/* 2>/dev/null | sort -h
```

Docker:

```bash
docker system df -v
```

Do not jump directly to `docker system prune -a --volumes`; identify what is consuming space first.

---

# Certificate/TLS failures

```bash
openssl s_client -connect <host>:443 -servername <host>
```

Dates/issuer:

```bash
openssl s_client -connect <host>:443 -servername <host> </dev/null 2>/dev/null \
  | openssl x509 -noout -subject -issuer -dates
```

Also check:

- hostname/SAN mismatch
- expired cert
- incomplete chain
- proxy interception
- local clock skew
- private CA trust

---

# What to capture before restarting something

Whenever practical:

```text
timestamp
environment/account/context
resource name
current status
recent events
last ~200 log lines
exit/restart reason
resource usage
recent deployment/config change
dependency health
```

A restart can erase the evidence that explains the failure.
