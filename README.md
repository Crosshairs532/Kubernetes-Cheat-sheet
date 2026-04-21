# Kubernetes-Cheat-sheet

 
## Commands
 
### Cluster & Minikube
 
```bash
minikube start --driver=docker          # Start local cluster
minikube start --driver=virtualbox
minikube dashboard                      # Open web UI
minikube service <service-name>         # Access service locally
```
 
---
 
### Pods
 
```bash
kubectl get pods                        # List all pods
kubectl get pods -o wide                # With node/IP details
kubectl get pods <pod-name> --show-labels
kubectl describe pod <pod-name>         # Full details + events
kubectl edit pod <pod-name>             # Edit live pod config
kubectl exec -it <pod-name> -- sh       # Shell into pod
kubectl delete pods --all               # Delete every pod
```
 
**Create a pod (quick test):**
```bash
kubectl run <pod-name> --image=nginx
```
 
**Dry run — preview without creating:**
```bash
kubectl run <pod-name> --dry-run=client
kubectl run <pod-name> --dry-run=client -o yaml > new.yaml   # Save to file
```
 
---
 
### Deployments
 
```bash
kubectl get deployments
kubectl get all                         # Everything at once
 
# Create
kubectl create deployment <name> --image=<image>
 
# Scale
kubectl scale deployment/<name> --replicas=3
 
# Update image
kubectl set image deployment/<name> <container-name>=<new-image>
 
# Delete
kubectl delete deployment <name>
```
 
---
 
### Rollouts
 
```bash
# Check status
kubectl rollout status deployment/<name>
 
# See history
kubectl rollout history deployment/<name>
kubectl rollout history deployment/<name> --revision=1
 
# Undo last rollout
kubectl rollout undo deployment/<name>
 
# Go back to specific version
kubectl rollout undo deployment/<name> --to-revision=2
```
 
---
 
### Services
 
```bash
# Expose a deployment as a service
kubectl expose deployment <name> --type=NodePort --port=80
kubectl expose deployment <name> --type=LoadBalancer --port=80
 
# Access service via minikube
minikube service <service-name>
```
 
---
 
### ReplicationController
 
```bash
kubectl get rc
kubectl scale rc <name> --replicas=0    # Scale down to 0
kubectl scale rc <name> --replicas=3
kubectl delete rc <name>
```
 
---

### NameSpaces
####  Commonly Used 

```bash
# List all namespaces
kubectl get namespaces
kubectl get ns                          # short alias

# Create a namespace
kubectl create namespace namespace-1
kubectl create ns namespace-1           # short alias

# Delete a namespace (deletes EVERYTHING inside it)
kubectl delete namespace namespace-1

# Describe a namespace
kubectl describe namespace namespace-1
```

#### Working inside a namespace
```bash
# The -n flag targets a specific namespace
kubectl get pods -n namespace-1
kubectl get deployments -n namespace-1
kubectl get services -n namespace-1
kubectl get all -n namespace-1          # everything in ns

# See resources across ALL namespaces
kubectl get pods --all-namespaces
kubectl get pods -A                    # short alias

# exec into a pod in a specific namespace
kubectl exec -it pod-name -n namespace-1 -- sh

```
#### Set default namespace for current context
```bash 
kubectl config set-context --current \
  --namespace=namespace-1

# Now kubectl get pods finds pods in namespace-1 without -n

# Check what your current context is
kubectl config get-contexts
kubectl config current-context
```
### Declarative (File-based) — The Professional Way
 
```bash
# Apply config
kubectl apply -f deployment.yaml
 
# Apply multiple files
kubectl apply -f deployment.yaml -f service.yaml
 
# Delete using file
kubectl delete -f deployment.yaml
kubectl delete -f deployment.yaml -f service.yaml
```
 
---
 
##  kubectl run vs create deployment vs apply
 
| Command | What it creates | Pod dies? | Best for |
|---|---|---|---|
| `kubectl run` | Single Pod only | Stays dead | Quick one-off tests |
| `kubectl create deployment` | Deployment + RS + Pods | Auto-restarted | Real projects (imperative) |
| `kubectl apply -f file.yaml` | Whatever the file says | Depends on config | Production (declarative) |
 
---
 
## Imperative vs Declarative
 
| | Imperative | Declarative |
|---|---|---|
| How | Run commands step by step | Write YAML, apply once |
| Example | `kubectl create deployment...` | `kubectl apply -f app.yaml` |
| Good for | Quick tests | Production, version control |
| Repeatable? | ❌ Manual every time | ✅ Apply same file anytime |
 
---
 
##  Liveness Probe
 
Kubernetes checks if your app is alive. If the probe fails, the container restarts.
 
```yaml
livenessProbe:
  httpGet:
    path: /        # Hit this endpoint
    port: 8080
  initialDelaySeconds: 5    # Wait 5s before first check
  periodSeconds: 10         # Check every 10s
```
 
---
 
##  Docker + Kubernetes Workflow
 
```bash
# 1. Tag your local image for Docker Hub
docker tag <current-image> <dockerhub-username>/<image-name>
 
# 2. Push to Docker Hub
docker push <dockerhub-username>/<image-name>
 
# 3. Deploy to Kubernetes
kubectl create deployment <name> --image=<dockerhub-username>/<image-name>
 
# 4. Update running deployment with new image
kubectl set image deployment/<name> <container>=<dockerhub-username>/<image-name>
```
 
---
 
##  Quick Reference Card
 
```
OBJECT          COMMAND PREFIX              EXAMPLE
──────────────────────────────────────────────────────
Pod             kubectl get pods            kubectl describe pod my-pod
Deployment      kubectl get deployments     kubectl scale deployment/app --replicas=3
Service         kubectl get services        kubectl expose deployment app --type=LoadBalancer
RC              kubectl get rc              kubectl scale rc main --replicas=0
All             kubectl get all
```
