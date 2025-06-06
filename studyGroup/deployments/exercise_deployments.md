# Deployments

## Doc

https://kubernetes.io/docs/concepts/workloads/controllers/deployment/
https://kubernetes.io/docs/tutorials/kubernetes-basics/deploy-app/deploy-intro/

## Objective

The goal is to learn how to work with Kubernetes Deployments, understand their benefits over raw Pods, and practice common deployment operations like scaling and updates.

## Exercise 1: Working with Deployments

Create a Deployment with the following specifications:
- Name: nginx-deployment
- Namespace: ckad
- 3 replicas
- Image: nginx:1.25.1
- Labels: app=nginx, environment=dev
- Resource limits: 200m CPU, 256Mi memory
- Resource requests: 100m CPU, 128Mi memory

After creating the deployment:
1. Scale the deployment to 5 replicas
2. Update the image to nginx:1.25.2
3. Rollback to the previous version
4. Check the deployment history

# Solution Steps

### 1. Create the Deployment

First, switch to the kind cluster context:

```bash
kubectl config get-contexts
kubectl config use-context kind-k8s-labs
```

Create the YAML manifest:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: ckad
  labels:
    app: nginx
    environment: dev
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
        environment: dev
    spec:
      containers:
      - name: nginx
        image: nginx:1.25.1
        ports:
        - containerPort: 80
        resources:
          limits:
            cpu: 200m
            memory: 256Mi
          requests:
            cpu: 100m
            memory: 128Mi
```

### 2. Apply the Manifest

```bash
kubectl apply -f deployment.yaml
```

### 3. Check the Deployment Status

```bash
kubectl get deployments -n ckad
kubectl get pods -n ckad -l app=nginx
```

### 4. Scale the Deployment

```bash
kubectl scale deployment nginx-deployment -n ckad --replicas=5
```

Verify the scaling:

```bash
kubectl get deployments -n ckad
kubectl get pods -n ckad -l app=nginx
```

### 5. Update the Image

```bash
kubectl set image deployment/nginx-deployment -n ckad nginx=nginx:1.25.2
```

Check the rollout status:

```bash
kubectl rollout status deployment/nginx-deployment -n ckad
```

### 6. View Rollout History

```bash
kubectl rollout history deployment/nginx-deployment -n ckad
```

### 7. Rollback to Previous Version

```bash
kubectl rollout undo deployment/nginx-deployment -n ckad
```

Verify the rollback:

```bash
kubectl rollout status deployment/nginx-deployment -n ckad
kubectl describe deployment nginx-deployment -n ckad
```

## Exercise 2: Advanced Deployment Strategies

Create a Deployment that uses a RollingUpdate strategy with:
- maxSurge: 1
- maxUnavailable: 0

This will ensure zero-downtime updates by creating one new pod before removing an old one.

## Solution Steps

### 1. Create the Deployment with RollingUpdate Strategy

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: zero-downtime-nginx
  namespace: ckad
  labels:
    app: nginx
    environment: prod
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
        environment: prod
    spec:
      containers:
      - name: nginx
        image: nginx:1.25.1
        ports:
        - containerPort: 80
```

Apply the manifest:

```bash
kubectl apply -f zero-downtime-deployment.yaml
```

### 2. Perform an Update and Observe the Rolling Update Process

```bash
kubectl set image deployment/zero-downtime-nginx -n ckad nginx=nginx:1.25.2
```

Watch the pods during the update:

```bash
kubectl get pods -n ckad -l app=nginx -w
```

## Exercise 3: Deployment Management

## Doc

https://kubernetes.io/docs/concepts/workloads/controllers/deployment/
https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#updating-a-deployment

## Objective

The goal is to practice creating, updating, scaling, and rolling back deployments in Kubernetes.

## Exercise

Create a Deployment named nginx with 3 replicas. The Pods should use the nginx:1.23.0 image and the name nginx. The Deployment uses the label tier=backend. The Pod template should use the label app=v1.

List the Deployment and ensure that the correct number of replicas is running.

Update the image to nginx:1.23.4.

Verify that the change has been rolled out to all replicas.

Assign the change cause "Pick up patch version" to the revision.

Scale the Deployment to 5 replicas.

Have a look at the Deployment rollout history. Revert the Deployment to revision 1.

Ensure that the Pods use the image nginx:1.23.0.

## Solution Steps

### 1. Create the Deployment

First, switch to the kind cluster context:

```bash
kubectl config get-contexts
kubectl config use-context kind-k8s-labs
```

Create the YAML manifest:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels:
    tier: backend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: v1
  template:
    metadata:
      labels:
        app: v1
    spec:
      containers:
      - name: nginx
        image: nginx:1.23.0
        ports:
        - containerPort: 80
```

### 2. Apply the Manifest

```bash
kubectl apply -f nginx-deployment-exercise.yaml
```

### 3. Check the Deployment Status

```bash
kubectl get deployments
kubectl get pods -l app=v1
```

### 4. Update the Image

```bash
kubectl set image deployment/nginx nginx=nginx:1.23.4
```

Check the rollout status:

```bash
kubectl rollout status deployment/nginx
```

### 5. Assign a Change Cause

```bash
kubectl annotate deployment/nginx kubernetes.io/change-cause="Pick up patch version"
```

### 6. Scale the Deployment

```bash
kubectl scale deployment nginx --replicas=5
```

Verify the scaling:

```bash
kubectl get deployments
kubectl get pods -l app=v1
```

### 7. View Rollout History

```bash
kubectl rollout history deployment/nginx
```

### 8. Rollback to Revision 1

```bash
kubectl rollout undo deployment/nginx --to-revision=1
```

Verify the rollback:

```bash
kubectl rollout status deployment/nginx
kubectl describe deployment nginx
kubectl get pods -l app=v1 -o jsonpath='{.items[0].spec.containers[0].image}'
```

This should confirm that the image has been reverted back to nginx:1.23.0.



## Exercise 4: Deployment Strategies

One of your teammates created a Deployment YAML manifest to operate the container image grafana/grafana:9.5.9.
Create the Deployment object from the YAML manifest file deployment-grafana.yaml:


```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grafana
  namespace: ckad
spec:
  replicas: 6
  selector:
    matchLabels:
      app: grafana
  template:
    metadata:
      labels:
        app: grafana
    spec:
      containers:
        - image: grafana/grafana:9.5.9
          name: grafana
          ports:
            - containerPort: 3000
```

You need to update all replicas with the container image grafana/grafana:10.1.2. 
Make sure that the rollout happens in batches of two replicas at a time. Ensure that a readiness probe is defined.


## Exercise 5. blue-green Deployment

In this exercise, you will set up a blue-green Deployment scenario. You'll first create the initial (blue) Deployment and expose it with a Service. Later, you will create a second (green) Deployment and switch over traffic.

Create a Deployment named nginx-blue with 3 replicas. The Pod template of the Deployment should use container image nginx:1.23.0 and assign the label version=blue.

Expose the Deployment with a Service of type ClusterIP named nginx. Map the incoming and outgoing port to 80. Select the Pod with label version=blue.

Run a temporary Pod with the container image alpine/curl:3.14 to make a call against the Service using curl.

Create a second Deployment named nginx-green with 3 replicas. The Pod template of the Deployment should use container image nginx:1.23.4 and assign the label version=green.

Change the Service's label selection so that traffic will be routed to the Pods controlled by the Deployment nginx-green.

Delete the Deployment named nginx-blue.

Run a temporary Pod with the container image alpine/curl:3.14 to make a call against the Service.

