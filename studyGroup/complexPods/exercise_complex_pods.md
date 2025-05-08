# Exercise 1: Working with init containers

## Doc

https://kubernetes.io/docs/concepts/storage/volumes/
https://kubernetes.io/docs/concepts/storage/volumes/#emptydir:~:text=to%20learn%20more.-,emptyDir,-For%20a%20Pod

## Objective

The objective is to complex-pods

# Exercise

Create a YAML manifest for a Pod named complex-pod. The main application container named app should use the image
nginx:1.25.1 and expose the container port 80. Modify the YAML manifest so that the Pod defines an init container
named setup that uses the image busybox:1.36.1. The init container runs the command wget -O- google.com.
Create the Pod from the YAML manifest.
Download the logs of the init container. You should see the output of the wget command.
Open an interactive shell to the main application container and run the ls command. Exit out of the container.
Force-delete the Pod.

## Solution Steps

### 1. Create the Pod Manifest

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
  name: complex-pod
  namespace: ckad
  labels:
    role: complex-pod
spec:
  initContainers:
    - name: setup
      image: busybox:1.36.1
      command: ['wget', 'O', 'google.com']
  containers:
    - name: app
      image: nginx:1.25.1
      ports:
        - containerPort: 80
```

### 2. Apply the Manifest

```bash
kubectl apply -f complex-pod.yaml
```

### 3. Check the Pod Status

```bash
kubectl get pods -n ckad
``` 

### 4. Download the Logs of the Init Container

```bash
kubectl logs complex-pod -c setup -n ckad
``` 

### 5. Open an Interactive Shell to the Main Application Container

```bash
kubectl exec -it complex-pod -n ckad -- /bin/sh
```

### 6. Run the ls Command

```bash
ls
``` 
### 7. Exit the Container

```bash
exit
```

### 8. Force-Delete the Pod

```bash
kubectl delete pod complex-pod -n ckad --force --grace-period=0
```

Alternatively, you can use the following command to delete the pod:

```bash
kubectl delete --force -f complex-pod.yaml
```

### 9. Verify the Pod is Deleted

```bash
kubectl get pods -n ckad
```


# Exercise 2: Working with sidecar pattern


Create a YAML manifest for a Pod named data-exchange. The main application container named main-app should use the image busybox:1.36.1. The container runs a command that writes a new file every 30 seconds in an infinite loop in the directory /var/app/data. The filename follows 
the pattern {counter++}-data.txt. The variable counter is incremented every interval and starts with the value 1.

Modify the YAML manifest by adding a sidecar container named sidecar. The sidecar container uses the image busybox:1.36.1 and runs a command that counts the number of files produced by the main-app container every 60 seconds in an infinite loop. 
The command writes the number of files to standard output.

Define a Volume of type emptyDir. Mount the path /var/app/data for both containers.

Create the Pod. Tail the logs of the sidecar container.

Delete the Pod.

## 1. Create the Pod Manifest

The yaml will look like:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: data-exchange
  namespace: ckad
  labels:
    role: data-exchange
spec:
  containers:
    - name: main-app
      image: busybox:1.36.1
      command: ['sh', '-c', 'counter=1; while true; do echo $counter-data.txt > /var/app/data/$counter-data.txt; counter=$((counter+1)); sleep 30; done']
      volumeMounts:
        - name: data-volume
          mountPath: /var/app/data
    - name: sidecar
      image: busybox:1.36.1
      command: ['sh', '-c', 'while true; do ls /var/app/data | wc -l; sleep 60; done']
      volumeMounts:
        - name: data-volume
          mountPath: /var/app/data
  volumes: 
    - name: data-volume
      emptyDir: {}
```

### 2. Apply the Manifest

```bash
kubectl apply -f data-exchange.yaml
```

### 3. Check the Pod Status

```bash 
kubectl get pods -n ckad
```

### 4. Tail the Logs of the Sidecar Container

```bash
kubectl logs -f data-exchange -c sidecar -n ckad
```

### 5. Delete the Pod

```bash
kubectl delete pod data-exchange -n ckad --force --grace-period=0
```



