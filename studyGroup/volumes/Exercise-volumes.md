# Exercise 1: Working with Volumes in Pods

## Doc

https://kubernetes.io/docs/concepts/storage/volumes/
https://kubernetes.io/docs/concepts/storage/volumes/#emptydir:~:text=to%20learn%20more.-,emptyDir,-For%20a%20Pod

## Objective

The objective is to practise how volumes are accessed and shared between different containers.

# Exercise

* Create a Pod YAML manifest with two containers that use the image alpine:3.12.0. Provide a command for both containers that keep them running forever. 
* Define a Volume of type emptyDir for the Pod. Container 1 should mount the Volume to path /etc/a, and container 2 should mount the Volume to path /etc/b. 
* Open an interactive shell for container 1 and create the directory data in the mount path. Navigate to the directory and create the file hello.txt with the contents "Hello World." Exit out 
* Open an interactive shell for container 2 and navigate to the directory /etc/b/data. Inspect the contents of file hello.txt. Exit out of the container.


## Solution Steps

### 1. Create the Pod Manifest 
    
First, switch to the kind cluster context:

```bash
kubectl config get-contexts
kubectl config use-context kind-k8s-labs
```

Create the YAML manifest with two alpine containers that run forever. 
As usual, we can run the following command to generate a yaml file where we can start from. This time, we have dumped the output of the yaml into a file
```bash
kubectl run shared-volumes --image=alpine:3.12.0 --dry-run=client -o yaml -n ckad --command -- sh -c "while true; do sleep 3600; done" > shared-volumes.yaml
```

The yaml will look like:

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: shared-volumes
  name: shared-volumes
  namespace: ckad
spec:
  containers:
    - name: alpine-container-1
      image: alpine:3.12.0
      command: ["sh", "-c", "while true; do sleep 3600; done"]
    - name: alpine-container-2
      image: alpine:3.12.0
      command: ["sh", "-c", "while true; do sleep 3600; done"]
status: {}
```

### 2. Define teh volumes

We have to define a common volume and then create a path to each container
The YAML will look like: 
```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: shared-volumes
  name: shared-volumes
  namespace: ckad
spec:
  containers:
    - name: alpine-container-1
      image: alpine:3.12.0
      command: ["sh", "-c", "while true; do sleep 3600; done"]
      volumeMounts:
        - mountPath: /etc/a
          name: shared-volume
    - name: alpine-container-2
      image: alpine:3.12.0
      command: ["sh", "-c", "while true; do sleep 3600; done"]
      volumeMounts:
        - mountPath: /etc/b
          name: shared-volume
  volumes:
    - name: shared-volume
      emptyDir: {}
status: {}
```

### 3. Create data in container 1 inside data directory

First, we can create the pods 

```bash
kubectl apply -f shared-volumes.yaml
```

To do this, we have to use ``exec``in interactable way command as follows: 

```bash
kubectl exec -it shared-volumes -c alpine-container-1 -n ckad -- sh
```

Once inside the pod, you can create the directory data and the txt
```bash
mkdir -p /etc/a/data
cd /etc/a/data
echo "Hello World." > hello.txt
```


### 4. Verify data from shared volume is accesible from container 2

```bash
kubectl exec -it shared-volumes -c alpine-container-2 -n ckad -- sh
```

# Inside container 2:
```bash
cd /etc/b/data
cat hello.txt
exit
```

## Notes
- The `emptyDir` volume is created when a Pod is assigned to a node and exists as long as that Pod is running on that node
- Both containers mount the same volume at different paths (`/etc/a` and `/etc/b`)
- The `sleep 3600` command keeps the containers running indefinitely (1 hour)
- Data written to the volume by one container is immediately available to the other container
- The `emptyDir` volume is initially empty and is created when the Pod is assigned to a node



# Exercise 2: Working with persisten Volumes

## Doc
https://kubernetes.io/docs/concepts/storage/persistent-volumes/

## Objective
The objective is to learn how to work with persistent volumes that are kept even though after the pods have been deleted 
* Create a Persistent Volume (PV) with hostPath so that the data is stored in the node's file system
* Create a Persistent Volume Claim (PVC) that will request storage from PV.
* Mount the PVC in a pod with nginx
* Check the data is persisted after the pod has been terminated. 

## Exercise:
Create a PersistentVolume named logs-pv that maps to the hostPath /var/logs. The access mode should be ReadWriteOnce and ReadOnlyMany. 
Provision a storage capacity of 5Gi. Ensure that the status of the PersistentVolume shows Available.

Create a PersistentVolumeClaim named logs-pvc. It uses ReadWriteOnce access. Request a capacity of 2Gi. Ensure that the status of the 
PersistentVolume shows Bound.

Mount the PersistentVolumeClaim in a Pod running the image nginx at the mount path /var/log/nginx.

Open an interactive shell to the container and create a new file named my-nginx.log in /var/log/nginx. Exit out of the Pod.
Delete the Pod and re-create it with the same YAML manifest. Open an interactive shell to the Pod, navigate to the directory /var/log/nginx, 
and find the file you created before.


## Solution Steps


### 1. Create Persistent Volume

After checking kubectl help, we noticed we can't create a persistent volume with a command. So, it needs to be created
using a YAML manifest. We added Retain as persistentVolumeReclaimPolicy to keep the data after it finishes. 

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: logs-pv
  namespace: ckad
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadOnlyMany
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /var/logs
``` 

Create the persistent volume and check it's available

```bash
 kubectl apply -f logs-pv.yaml
```

```bash
kubectl get pv -n ckad
```

Output:
```
NAME      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
logs-pv   5Gi        RWO,ROX        Retain           Available                          <unset>                          9s
```

### 2. Create the persistent volume claim

It includes some lines from the official documentation that are not used in this exercise. It is just for information

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: logs-pvc
  namespace: ckad
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 2Gi
#  storageClassName: slow
#  selector:
#    matchLabels:
#      release: "stable"
#    matchExpressions:
#      - {key: environment, operator: In, values: [dev]}
```


```bash
 kubectl apply -f logs-pvc.yaml
```

Check PVC

```bash
kubectl get pv -n ckad
```
Output:
```
NAME       STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
logs-pvc   Pending                                      standard       <unset>                 2m43s
```

We can describe the pvc to see why it is in pending status and we can check that the reason is because
it's waiting for first consumer to be created before binding


```bash 
kubectl describe pvc logs-pvc -n ckad
```


We can do two things. 
1. To force to bind the PVC to teh PV using storageClassName: standard in both PV and PVC

doc -> https://kubernetes.io/docs/concepts/storage/storage-classes/#:~:text=for%20seamless%20migration.-,You,-can%20create%20a
Once storageClassName: "" is defined, the PVC is bound to the PV

```bash
kubectl get pvc logs-pvc -n ckad
```

Note that the status is "bound"

```
NAME       STATUS   VOLUME    CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
logs-pvc   Bound    logs-pv   5Gi        RWO,ROX                       <unset>                 3s
```


2. Alternatively, the PVC is bound when the pod is deployed

See point 2






## 2. Create a pod that mounts the PVC and it will bind them automatically. 

We create the YAML file based on a nginx image 

```bash
kubectl run nginx-pod --image=nginx --dry-run=client -o yaml -n ckad > nginx-pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx-pod
  name: nginx-pod
  namespace: ckad
spec:
  containers:
  - image: nginx
    name: nginx-pod
    resources: {}
    volumeMounts:
      - mountPath: /var/log/nginx
        name: logs-storage
  volumes:
    - name: logs-storage
      persistentVolumeClaim:
        claimName: logs-pvc

  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

We can add the volume and then run the manifest

```bash
kubectl apply -f nginx-pod.yaml 
```

Right after the pod is created, the PVC is bound to the Persistent volume

```bash
kubectl get pvc logs-pvc -n ckad
```


## 3. Check the logs are persisted in the volume after the pod has been deleted. 


We need to create something in the log directory

```bash 
kubectl exec -it nginx-pod -n ckad -- /bin/bash
```

Then, inside the nginx pod we create some logs and exit the pod

```bash 
cd var/log/nginx
echo "My logs" > my-nginx.log
exit
```

We can delete the pod and recreate it to check that the previous log is still there

```bash
kubectl delete pod nginx-pod -n ckad
```

```bash
kubectl apply -f nginx-pod.yaml -n ckad
```

```bash 
kubectl exec -it nginx-pod -n ckad -- /bin/bash
```


```bash 
cat var/log/nginx/my-nginx.log
```