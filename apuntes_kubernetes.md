
 

 ``kubectl get pods --sort-by=.status.containerStatuses[0].restartCount``

CREAR PODS, DEPLOYMENTS, SERVICES, ETC.

Se pueden lanzar o crear pods, deployments, services, etc. con un archivo de configuración en yaml. Para ello se utiliza el comando apply.

 ``kubectl apply -f fileName.yaml``
Normalmente lo que hacemos es lanzar un deployment que levanta un pod


Differences between limits and requests in resources are that limits are the maximum amount of resources that a container can use, 
and requests are the minimum amount of resources that a container needs to run.

Kubernetes va a tratar de mantener tantos pods como estén definidos en el deployment. Si un pod muere, kubernetes lo vuelve a levantar.
Por ejemplo si se define un deployment con 3 replicas, kubernetes va a tratar de mantener 3 pods siempre levantados. Si borro un pod a mano con kubeclt delete pod podName, kubernetes lo vuelve a levantar.


DemonSets se utulizan para levantar un pod en cada nodo del cluster. Por ejemplo para levantar un pod que recolecte logs de cada nodo o para taeas de monitoreo.

También se pueden usar statefulSets para levantar pods que necesiten un almacenamiento persistente. Por ejemplo para bases de datos.
El pod estará atado a un volumen. 
En el ejemplo de 05-statefulSet.yaml se crea un statefulSet con un volumen. 
Como vemos en el archivo yaml, se crea un spec dentro del spec del statefulset donde definimos el pod llamado my-frontend que correrá una imagen busybox y montará el volumen en /data. 
Le pasamos dos argumentos -sleep e -infinity a esa imagen.   
El volumen se define en volumeClaimTemplates donde le decimos el tipo de almacenamiento, acceso, el tamaño y el nombre del volumen.
StorageClassName se utiliza para definir el tipo de almacenamiento. Es como un driver para Kubernetes para usar un almacenamiento que puede ser en disco, 
en la nube, etc. Al usar do-block-storage se está utilizando un disco en digital ocean. Podríamos usar otro tipo de almacenamiento como por ejemplo un disco en AWS.
Para alamcenamiento en minikube se puede usar storageClassName: standard.


Pod Netwworking 

Cada pod tiene su propia dirección IP, pero cada contenedor dentro de un pod comparte la misma dirección IP.
La información sobre enrutamiento se almacena en una tabla de enrutamiento llamado etcd donde se indica que dirección IP se encuentra en cada nodo y como 
llegar hasta cada Pod



Servicios 
Son una forma de contactar aplicaciones que corren en un cluster de Kubernetes ya sean desde fuera o dentro del cluster.
Los servicios en Kubernetes son set de pods. 
* Cluster IP (no hace falta especificarlo)
* Node port 
* Load balancer. Este tipo de servicio crea un balanceador de carga en la nube. Por ejemplo en AWS crea un balanceador de carga en AWS.


Caso de CLUSTER IP con un pequeño load balancer

En el 07-hello-deployment-svc-clusterIP.yamlxss se crear un deployment con una imagen de una app de Google de hello-app. 
Se crea un servicio de tipo ClusterIP que se va a encargar de enrutar el tráfico a los pods del deployment.
Como vemos, el servicio tiene un selector que apunta a la etiqueta app: hello-app que ha sido marcado con una etiqueta label en el deployment.
Como los pods cuando mueren y se reinician están cambiando de IP constantemente no podemos estar cambiando la IP cada vez. Para ello, 
se crea un servicio que enruta el tráfico a los pods del deployment.

Caso NodePort
Crear un servicio que escuche en un puerto en cada nodo y enrute el tráfico a los pods del deployment. Encuentra los pods basados en etiquetas 
y crea un puerto en cada nodo que enruta el tráfico a los pods.

Load Balancer
Lo mismo que NodePort pero crea un balanceador de carga en la nube. Por ejemplo en AWS crea un balanceador de carga en AWS.



INGRESS

Es un recurso que nos permite crear accesos a nuestros servicios basados en 
el path. 



Tengo que ver como instalar el ingress 11 con Helm. Después jugar con el trafico de red y ver como se enruta el tráfico a los servicios y pods.
Se definen unos paths que enrutan el tráfico a los servicios.


Configmap 
Es un recurso que nos permite almacenar configuraciones de nuestras aplicaciones.
Por ejemplo, si tenemos una aplicación que necesita una configuración de base de datos,
en lugar de tener que cambiar el código de la aplicación, podemos tener un configmap que
almacene la configuración de la base de datos y que la aplicación lea esa configuración de un archivo
de configuración.




## GET and INFO COMMANDS

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

| Command                                | Description                                                                           |
|----------------------------------------|---------------------------------------------------------------------------------------|
| `kubectl apply -f fileName.yaml`       | Create a resource for a pod, deployment, service, etc, from a file                    |
| `kubectl delete resource resourceName` | Delete a resource: pod, deployment, statefulsets.. e.g.: `kubectl delete pod podName` |   
| `kubectl delete -f fileName.yaml`      | Delete a resource: pod, deployment, statefulsets.. using a file name                  |
| `kubectl exec podName -- command`      | Execute a command withing a pod e.g.: `kubectl exec nginx -- sh`                      |




## DEBUGGING COMMANDS

| Command                                 | Description                                      |
|-----------------------------------------|--------------------------------------------------|
| `kubectl logs podName`                  | Get logs from a pod                              |
| `kubectl logs podName -c containerName` | Get logs from a pod with multiple containers     |
| `kubectl logs podName --previous`       | Get logs from a pod that has been restarted      |
| `kubectl logs podName -f`               | Get logs from a pod and keep the connection open |
| `kubectl describe pod podName`          | Describe a pod and all its events                |
| `kubectl get events`                    | Get all events                                   |