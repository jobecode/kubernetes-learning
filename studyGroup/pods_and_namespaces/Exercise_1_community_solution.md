Create a new Pod named nginx running the image nginx:1.17.10. Expose the container port 80. The Pod should live in the namespace named ckad.
----------------------------------------------------------------------------


## Before starting the exercise make sure you are in the right context

```bash
kubectl config current-context
```

List the contexts available:

```bash
kubectl config get-contexts
```

If the output is not the right context, change it using the following command:

```bash
kubectl config use-context kind-k8s-labs
```


# Solution:

* It creates the namespace `ckad` or add --dry-run=client -o yaml to get the yaml file (it doesn't create the namespace)

```bash
kubectl create namespace ckad --dry-run=client -o yaml
```

* Output:

```yaml 
apiVersion: v1
kind: Namespace
metadata:
  name: ckad
```

Remove --dry-run=client -o yaml to effective create the pod
----------------------------------------------------

```bash
kubectl create namespace ckad
```

It creates the pod called nginx in the namespace ckad. --dry-run=client -o yaml can be added to get the yaml file (it doesn't create the pod)
----------------------------------------------------

```bash
kubectl run nginx --image=nginx:1.17.10 --port=80 -n ckad
```

º Output:

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



Get the details of the Pod including its IP address.
----------------------------------------------------

Solution (two options):
* With -o wide flag to get the IP address of the Pod:

```bash
kubectl get pod nginx -n ckad -o wide
```

* With describe command to get the IP address of the Pod:

```bash
kubectl describe pod nginx -n ckad
```

Create a temporary Pod that uses the busybox:1.36.1 image to execute a wget command inside of the container. 
The wget command should access the endpoint exposed by the nginx container. You should see the HTML response body rendered in the terminal.
Here we can see the pod nginx is reachable inside the namespace ckad.
---------------------------------------------------------------------------------

Solution:

* Enter the busybox container and then execute the wget command:

```bash
kubectl run busybox --image=busybox:1.36.1 -n ckad -it --rm
```

* Alternatively, you can run the command directly: Restarts=Never is used to avoid the pod to restart after the command is executed.

```bash
kubectl run busybox --image=busybox:1.36.1 -n ckad -it --rm --restart=Never -- wget -O index.html http://10.244.0.5
```


Get the logs of the nginx container.
------------------------------------

Solution:

```bash
kubectl logs nginx -n ckad
```

* Also, logs from busybox can be shown

```bash
kubectl logs busybox -n ckad
```



Add the environment variables DB_URL=postgresql://mydb:5432 and DB_USERNAME=admin to the container of the nginx Pod.
---------------------------------------------------------------------------------

Solution: The pod can be edited using the kubectl edit command. It can use vim, nano or code as the editor.
However, with edit command, the environment variables cannot be added. The pod should be deleted and recreated with the environment variables.

```bash
KUBE_EDITOR="nano" kubectl edit pod nginx -n ckad
```

* Create the pod nginx with the environment variables: As always dry-run=client -o yaml can be added to get the yaml file (it doesn't create the pod)

```bash
kubectl run nginx --image=nginx:1.17.10 --port=80 -n ckad --dry-run=client -o yaml --env="DB_URL=postgresql://mydb:5432" --env="DB_USERNAME=admin"
```

Open a shell for the nginx container and inspect the contents of the current directory ls -l. Exit out of the container.
---------------------------------------------------------------------------------

Solution:

```bash
kubectl exec nginx -n ckad -it -- /bin/bash
```
* Inside the pod run: 

```bash
ls -l
```
* And print the environment variables:

```bash
printenv
```

* To exit the container:

```bash
exit
```

Extra: Make a port-forward to the nginx container and access the nginx welcome page from your local machine.
---------------------------------------------------------------------------------

Solution:

```bash
kubectl port-forward pod/nginx 8080:80 -n ckad
```