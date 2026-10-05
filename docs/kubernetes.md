# Kubernetes / kubectl Cheat Sheet

## Contexts and namespaces

```bash
kubectl config get-contexts
kubectl config current-context
kubectl config use-context <context>

kubectl get ns
kubectl config view --minify --output 'jsonpath={..namespace}'; echo
kubectl config set-context --current --namespace=<namespace>
```

Prefer explicit targeting in unfamiliar environments:

```bash
kubectl --context <context> -n <namespace> get pods
```

## Discover resources

```bash
kubectl get pods
kubectl get pods -A
kubectl get pods -o wide
kubectl get all -n <namespace>

kubectl get deploy
kubectl get svc
kubectl get ingress
kubectl get configmap
kubectl get secret
kubectl get pvc
kubectl get nodes
```

Watch:

```bash
kubectl get pods -w
kubectl get deploy -w
```

## Describe

```bash
kubectl describe pod <pod>
kubectl describe deploy <deployment>
kubectl describe node <node>
```

## Logs

```bash
kubectl logs <pod>
kubectl logs -f <pod>
kubectl logs --tail=200 <pod>
kubectl logs --since=10m <pod>
kubectl logs <pod> -c <container>
kubectl logs -f <pod> -c <container>
```

Logs from the previous crashed container instance:

```bash
kubectl logs <pod> -c <container> --previous
```

All containers in a pod:

```bash
kubectl logs <pod> --all-containers=true
```

## Exec into a pod

```bash
kubectl exec -it <pod> -- bash
kubectl exec -it <pod> -- sh
kubectl exec -it <pod> -c <container> -- sh
kubectl exec <pod> -- env
kubectl exec <pod> -- ps aux
```

## Copy files

```bash
kubectl cp <namespace>/<pod>:/path/file ./file
kubectl cp ./file <namespace>/<pod>:/path/file
```

Specify a container:

```bash
kubectl cp ./file <namespace>/<pod>:/tmp/file -c <container>
```

## Port forwarding

```bash
kubectl port-forward pod/<pod> 8080:80
kubectl port-forward svc/<service> 8080:80
kubectl port-forward deploy/<deployment> 8080:8080
```

Bind beyond localhost only when you mean to:

```bash
kubectl port-forward --address 0.0.0.0 pod/<pod> 8080:80
```

## Resource usage

Requires Metrics Server:

```bash
kubectl top nodes
kubectl top pods
kubectl top pods -A
kubectl top pod <pod> --containers
```

## Deployments / rollouts

```bash
kubectl rollout status deploy/<deployment>
kubectl rollout history deploy/<deployment>
kubectl rollout restart deploy/<deployment>
kubectl rollout undo deploy/<deployment>
kubectl rollout undo deploy/<deployment> --to-revision=<n>
```

Scale:

```bash
kubectl scale deploy/<deployment> --replicas=3
```

## Events

```bash
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl get events -n <namespace> --sort-by=.metadata.creationTimestamp
kubectl get events --field-selector involvedObject.name=<pod>
```

## YAML

```bash
kubectl get pod <pod> -o yaml
kubectl get deploy <deployment> -o yaml
kubectl get svc <service> -o yaml
```

Apply:

```bash
kubectl apply -f file.yaml
kubectl diff -f file.yaml
```

Delete:

```bash
kubectl delete -f file.yaml
kubectl delete pod <pod>                 # recreates if managed by controller
```

## Labels/selectors

```bash
kubectl get pods --show-labels
kubectl get pods -l app=<app>
kubectl get pods -l 'environment in (prod,staging)'
kubectl label pod <pod> key=value
```

## JSONPath / compact queries

Pod names:

```bash
kubectl get pods -o jsonpath='{.items[*].metadata.name}'; echo
```

Images:

```bash
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.containers[*].image}{"\n"}{end}'
```

Node + pod:

```bash
kubectl get pods -o custom-columns='POD:.metadata.name,NODE:.spec.nodeName,STATUS:.status.phase'
```

## Wait for state

```bash
kubectl wait --for=condition=Ready pod/<pod> --timeout=120s
kubectl wait --for=condition=Available deploy/<deployment> --timeout=120s
```

## Authorization

```bash
kubectl auth can-i get pods
kubectl auth can-i create deployments
kubectl auth can-i '*' '*' --all-namespaces
```

Use the broad check carefully; it is diagnostic, not a request for more privileges.

## Debug ephemeral container

Useful when the application container is minimal/distroless:

```bash
kubectl debug -it pod/<pod> --image=busybox
```

Target a specific container's process namespace where supported:

```bash
kubectl debug -it pod/<pod> --image=busybox --target=<container>
```

## Service/DNS debugging

```bash
kubectl get svc <service> -o wide
kubectl get endpoints <service>
kubectl get endpointslice -l kubernetes.io/service-name=<service>
```

Temporary DNS/curl pod:

```bash
kubectl run tmp-shell --rm -it --restart=Never --image=busybox -- sh
```

Inside:

```sh
nslookup <service>
wget -S -O- http://<service>:<port>/
```

## Common failure triage

### CrashLoopBackOff

```bash
kubectl describe pod <pod>
kubectl logs <pod> --previous
kubectl get events --sort-by=.metadata.creationTimestamp
```

### ImagePullBackOff

```bash
kubectl describe pod <pod>
kubectl get events --field-selector involvedObject.name=<pod>
```

### Pending

```bash
kubectl describe pod <pod>
kubectl get nodes
kubectl top nodes
```

Look for scheduling errors, taints/tolerations, affinity, PVCs, and resource requests.

### Restart a managed workload

Prefer:

```bash
kubectl rollout restart deploy/<deployment>
```

Deleting a pod also causes most controllers to recreate it, but rollout restart is more explicit.
