
## GET and INFO COMMANDS

For more info run:

```bash
kubectl <command> --help
```

| Command                                                                | Description                                             |
|------------------------------------------------------------------------|---------------------------------------------------------|
| `kubectl get all`                                                      | Get all from namespace                                  |
| `kubectl get namespaces`                                               | Get all namespaces in a cluster                         |
| `kubectl get pods`                                                     | Get all pods from a namespace (default if not provided) |
| `kubectl get pod podName -o yaml`                                      | Get YAML with all its variables and parameters          |
| `kubectl get pod podName -o json`                                      | Get JSON with all its variables and parameters          |
| `kubectl get pod podName -o wide`                                      | Get pods with extended info                             |
| `kubectl get pvc`                                                      | Get Provisioning requests PVC                           |
| `kubectl get pods --sort-by=.status.containerStatuses[0].restartCount` | Get pods sorted by restart count                        |
| `kubectl get services`                                                 | Get all services                                        |
| `kubectl get nodes`                                                    | Get all nodes                                           |
| `kubectl get events`                                                   | Get all events                                          |
| `kubectl get deployments`                                              | Get all deployments                                     |
| `kubectl get statefulsets`                                             | Get all statefulsets                                    |
| `kubectl get daemonsets`                                               | Get all daemonsets                                      |
| `kubectl get jobs`                                                     | Get all jobs                                            |
| `kubectl get cronjobs`                                                 | Get all cronjobs                                        |
| `kubectl describe pod podName`                                         | Describe a pod and all its events                       |
| `kubectl get cluster-info`                                             | Display cluster information                             |
| `kubectl explain jobs`                                                 | Get all jobs                                            |
| `kubectl get cronjobs`                                                 | Get all cronjobs                                        |



## CONFIGURATION COMMANDS

| Command                                          | Description                                                                           |
|--------------------------------------------------|---------------------------------------------------------------------------------------|
| `kubectl apply -f fileName.yaml`                 | Create a resource for a pod, deployment, service, etc, from a file                    |
| `kubectl delete resource resourceName`           | Delete a resource: pod, deployment, statefulsets.. e.g.: `kubectl delete pod podName` |   
| `kubectl delete -f fileName.yaml`                | Delete a resource: pod, deployment, statefulsets.. using a file name                  |
| `kubectl exec podName -- command`                | Execute a command withing a pod e.g.: `kubectl exec nginx -- sh`                      |
| `kubectl port-forward pod/nginx 8080:80 -n ckad` | Port-forwarding from port 80 to 8080 (machine)                                        |



## DEBUGGING COMMANDS

| Command                                 | Description                                      |
|-----------------------------------------|--------------------------------------------------|
| `kubectl logs podName`                  | Get logs from a pod                              |
| `kubectl logs podName -c containerName` | Get logs from a pod with multiple containers     |
| `kubectl logs podName --previous`       | Get logs from a pod that has been restarted      |
| `kubectl logs podName -f`               | Get logs from a pod and keep the connection open |
| `kubectl describe pod podName`          | Describe a pod and all its events                |
| `kubectl get events`                    | Get all events                                   |