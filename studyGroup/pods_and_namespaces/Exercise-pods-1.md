# Exercise 1: Creating and Managing Pods

## Objective
Create a new Pod named `nginx` running the image `nginx:1.17.10`. Expose the container port 80. The Pod should live in the namespace named `ckad`.

## Prerequisites
Before starting the exercise, ensure you are in the correct context:

```bash
# Check current context
kubectl config current-context

# List available contexts
kubectl config get-contexts

# Switch to the correct context if needed
kubectl config use-context kind-k8s-labs
```

## Solution Steps

### 1. Create the Namespace
First, create the `ckad` namespace:

```bash
# Preview the namespace YAML (optional)
kubectl create namespace ckad --dry-run=client -o yaml

# Create the namespace
kubectl create namespace ckad
```

### 2. Create the Nginx Pod
Create the nginx pod in the `ckad` namespace:

```bash
# Create the pod
kubectl run nginx --image=nginx:1.17.10 --port=80 -n ckad
```

The pod specification will look like this:
```yaml
apiVersion: v1
kind: Pod
metadata:
    labels: 
      run: nginx
      name: nginx
      namespace: ckad
spec:
  containers:
  - image: nginx:1.17.10
    name: nginx
    ports:
      - containerPort: 80
    restartPolicy: Always
```

### 3. Verify Pod Details
Get the pod details including its IP address:

```bash
# Option 1: Using -o wide flag
kubectl get pod nginx -n ckad -o wide

# Option 2: Using describe command
kubectl describe pod nginx -n ckad
```

### 4. Test Pod Connectivity
Create a temporary busybox pod to test connectivity to the nginx pod:

```bash
# Option 1: Interactive shell
kubectl run busybox --image=busybox:1.36.1 -n ckad -it --rm

# Option 2: Direct command execution
kubectl run busybox --image=busybox:1.36.1 -n ckad -it --rm --restart=Never -- wget -O index.html http://10.244.0.5
```

### 5. View Container Logs
Check the logs of the containers:

```bash
# Nginx logs
kubectl logs nginx -n ckad

# Busybox logs (if still running)
kubectl logs busybox -n ckad
```

### 6. Add Environment Variables
Add environment variables to the nginx pod:

```bash
# Preview the pod YAML with new environment variables
kubectl run nginx --image=nginx:1.17.10 --port=80 -n ckad --dry-run=client -o yaml \
  --env="DB_URL=postgresql://mydb:5432" \
  --env="DB_USERNAME=admin"
```

### 7. Inspect Container
Access the container shell and inspect its contents:

```bash
# Open shell
kubectl exec nginx -n ckad -it -- /bin/bash

# Inside the container:
ls -l
printenv
exit
```

### 8. Port Forwarding (Optional)
Access the nginx welcome page from your local machine:

```bash
kubectl port-forward pod/nginx 8080:80 -n ckad
```

## Notes
- Use `--dry-run=client -o yaml` flag to preview YAML without creating resources
- The pod must be recreated to add environment variables as they cannot be modified in-place
- Port forwarding allows local access to container services