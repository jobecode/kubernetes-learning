
# Exercise 1: Working with labels

## Doc

https://kubernetes.io/docs/concepts/overview/working-with-objects/labels
https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#list-and-watch-filtering

## Objective

The objective is learn how to work with labels and annotations and to see the difference between them.



## Exercise

Create three Pods that use the image nginx:1.25.1. The names of the Pods should be pod-1, pod-2, and pod-3.

Assign the label tier=frontend to pod-1 and the label tier=backend to pod-2 and pod-3. All pods should also assign the label team=artemidis.

Assign the annotation with the key deployer to pod-1 and pod-3. Use your own name as the value.

From the command line, use label selection to find all Pods with the team artemidis or aircontrol and that are considered a backend service.


# Solution Steps

### 1. Create the Pods
First, switch to the kind cluster context:

```bash
kubectl config get-contexts
kubectl config use-context kind-k8s-labs
```

Create the YAML manifest.

The yaml will look like:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-1
  namespace: ckad
  labels:
    tier: frontend
    team: artemidis
  annotations:
    deployer: jobe
spec:
    containers:
        - name: app
          image: nginx:1.25.1
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-2
  namespace: ckad
  labels:
    tier: backend
    team: artemidis
spec:
    containers:
        - name: app
          image: nginx:1.25.1
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-3
  namespace: ckad
  labels:
    tier: backend
    team: artemidis
  annotations:
    deployer: jobe
spec:
    containers:
        - name: app
          image: nginx:1.25.1
```

### 2. Apply the Manifest

```bash
kubectl apply -f pods.yaml
```

### 3. Check the Pods Status

```bash
kubectl get pods -n ckad
```
### 4. Check the Labels and Annotations

```bash
kubectl get pods -n ckad --show-labels
```

### 5. Use Label Selection to Find Pods

```bash
kubectl get pods -n ckad -l team=artemidis,tier=backend
```

### 5.1 or using set-based requirements:

```bash
kubectl get pods -n ckad -l 'team in (artemidis), tier=backend'
```


### 6. Use Label Selection to Find Pods with Multiple Labels

```bash
kubectl get pods -n ckad -l 'team in (artemidis,aircontrol), tier=backend'
```

### 7. Use Label Selection to Find Pods with Annotations

```bash
kubectl get pods -n ckad -l 'deployer in (jobe)'
```

### 8. Use Label Selection to Find Pods with Annotations and Labels

```bash
kubectl get pods -n ckad -l 'deployer in (jobe), team=artemidis'
```

### 9. Use Label Selection to Find Pods with Annotations and Labels

```bash
kubectl get pods -n ckad -l 'deployer in (jobe), team=artemidis, tier=backend'
```

### 10. update the label of the pod-1

```bash
kubectl label pod pod-1 -n ckad tier=frontend --overwrite
```








## Exercise 2: Labels 

Create a Pod with the image nginx:1.25.1 that assigns two recommended labels: one for defining the application name with the value F5-nginx, and one for defining the tool used to manage the application named helm.

Render the assigned labels of the Pod object.



## Solution Steps

### 1. Create the Pod
First, switch to the kind cluster context:

```bash 
kubectl config get-contexts
kubectl config use-context kind-k8s-labs
```

Create the YAML manifest.
The yaml will look like:

Used the recommended labels from the documentation. https://kubernetes.io/docs/concepts/overview/working-with-objects/common-labels/

```yaml
apiVersion: v1  
kind: Pod
metadata:
  namespace: ckad
  labels:
    app.kubernetes.io/name: f5-nginx
    app.kubernetes.io/managed-by: helm
spec:
    containers:
        - name: app
          image: nginx:1.25.1
``` 

### 2. Apply the Manifest

```bash
kubectl apply -f pod.yaml
```
    
### 3. Check the Pod Status

```bash
kubectl get pods -n ckad
```
### 4. Check the Labels

```bash
kubetcl get pod f5-nginx -n ckad --show-labels
```

