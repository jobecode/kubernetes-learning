# Exercise 2: Working with Pod Manifests

## Objective
Create and manage a Pod using YAML manifests, demonstrating different container commands and pod lifecycle management.

## Solution Steps

### 1. Create a Pod with a Finite Loop
Create a Pod named `loop` that runs the `busybox:1.36.1` image and executes a command that prints a welcome message 10 times.

#### Generate the YAML Manifest
```bash
# Generate the YAML manifest
kubectl run loop --image=busybox:1.36.1 --dry-run=client -o yaml --restart=Never \
  --command -- sh -c 'for i in {1..10}; do echo "Welcome $i times"; done'
```

#### Create the YAML File
Save the following content to `loop-pod.yaml`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: loop
  namespace: ckad
spec:
  containers:
    - name: busybox
      image: busybox:1.36.1
      command: ["sh", "-c", "for i in {1..10}; do echo \"Welcome $i times\"; done"]
  restartPolicy: Never
```

#### Apply the Manifest and Check Status
```bash
# Apply the manifest
kubectl apply -f loop-pod.yaml

# Check pod status
kubectl get pod loop -n ckad

# Get detailed pod information
kubectl describe pod loop -n ckad
```

### 2. Modify the Pod for Continuous Date Output
Modify the Pod to run in an endless loop, printing the current date every second.

#### Delete the Existing Pod
```bash
# Delete the pod using the manifest
kubectl delete -f loop-pod.yaml
```

#### Create New YAML File
Save the following content to `loop_dates.yaml`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: loop
  namespace: ckad
spec:
  containers:
    - name: busybox
      image: busybox:1.36.1
      command: ["sh", "-c", "while true; do echo \"Current date: $(date)\"; sleep 1; done"]
  restartPolicy: Never
```

#### Apply the New Manifest
```bash
kubectl apply -f loop_dates.yaml
```

### 3. Inspect Pod Status and Events
Monitor the Pod's status and view its logs:

```bash
# Get detailed pod information including events
kubectl describe pod loop -n ckad

# View container logs
kubectl logs loop -n ckad
```

## Notes
- The `--dry-run=client -o yaml` flag is useful for generating YAML manifests without creating resources
- Pods with `restartPolicy: Never` will not restart after completion
- Use `kubectl logs` to view container output
- The `describe` command provides detailed information about pod events and status

## Community solution

