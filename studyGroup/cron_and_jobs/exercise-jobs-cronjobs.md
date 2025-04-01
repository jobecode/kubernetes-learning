# Kubernetes Jobs and CronJobs Exercise Solutions

##Doc
https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/
https://kubernetes.io/docs/concepts/workloads/controllers/job/

## Exercise 1: Random Hash Job

### Task
Create a Job named `random-hash` using the container image `alpine:3.17.3` that executes the shell command `echo $RANDOM | base64 | head -c 20`. Configure the Job to execute with two Pods in parallel. The number of completions should be set to 5.

### Solution
We'll use `batch/v1` that finishes after the job is done. The job is configured with:
- `completions: 5`: Total number of executions needed
- `parallelism: 2`: Maximum number of Pods running in parallel

Kubernetes jobs do not execute containers directly but create pods that contain the containers. We need to create a template inside the job that contains the container.

First, switch to the kind cluster context:

```bash
kubectl config get-contexts
kubectl config use-context kind-k8s-labs
```

Then apply the manifest file:

```bash
kubectl apply -f random-hash-job.yaml
```

## Exercise 2: Pod Analysis

### Task
Identify the Pods that executed the shell command. How many Pods do you expect to exist?

### Solution
You can list the Pods using any of these commands:

```bash
# List all pods in the namespace
kubectl get pods -n ckad

# List pods specifically for the random-hash job
kubectl get pods --selector=job-name=random-hash -n ckad

# List only completed pods
kubectl get pods --selector=job-name=random-hash -n ckad --field-selector=status.phase==Succeeded
```

Expected output:
```bash
NAME                READY   STATUS      RESTARTS   AGE
random-hash-56g7m   0/1     Completed   0          2m4s
random-hash-gch6l   0/1     Completed   0          118s
random-hash-kwwl6   0/1     Completed   0          117s
random-hash-sdhrk   0/1     Completed   0          115s
random-hash-zrbvn   0/1     Completed   0          2m4s
```

There are 5 Pods that executed the shell command, all in the Completed state. As expected, Kubernetes executed 2 Pods in parallel until the completions value of 5 was reached.

### Viewing Generated Hash
To see the logs from a specific pod:

```bash
kubectl logs random-hash-56g7m -n ckad
```

## Exercise 3: Job Deletion

### Task
Delete the Job and determine if the corresponding Pods continue to exist.

### Solution
```bash
kubectl delete job random-hash -n ckad
```

No, the Pods will not continue to exist because:
- The Pods have `restartPolicy: never`
- When the Job is deleted, its associated Pods are also deleted
- Unless configured with `ttlSecondsAfterFinished`, Pods are deleted after job completion

## Exercise 4: Google Ping CronJob

### Task
Create a new CronJob named `google-ping` that runs a curl command for google.com every two minutes.

### Solution
We can use either:
- `curlimages/curl:latest`: A lightweight image with curl pre-installed
- `alpine:3.17.3`: With curl installed via `apk add --no-cache curl`

Example container configuration:
```yaml
containers:
  - name: google-ping
    image: alpine:latest
    command: ["/bin/sh", "-c", "apk add --no-cache curl && curl -I https://www.google.com"]
```

### Monitoring Job Creation
Watch the creation of underlying Jobs:

```bash
kubectl get jobs -n ckad --watch
```

Expected output:
```bash
NAME                   STATUS     COMPLETIONS   DURATION   AGE
google-ping-29029972   Complete   1/1           4s         116s
google-ping-29029974   Running    0/1                      0s
google-ping-29029974   Running    0/1           0s         0s
google-ping-29029974   Running    0/1           4s         4s
google-ping-29029974   Complete   1/1           4s         4s
```

## Exercise 5: CronJob History Configuration

### Task
Reconfigure the CronJob to retain a history of seven executions.

### Solution
Add the following configuration to the CronJob spec:

```yaml
spec:
  successfulJobsHistoryLimit: 7
  failedJobsHistoryLimit: 7
```

Notes:
- Default values are 3 for successful jobs and 1 for failed jobs
- Setting to 0 will delete jobs immediately after completion
- This configuration will keep 7 successful and 7 failed jobs before deleting the oldest ones

## Exercise 6: Concurrent Execution Control

### Task
Reconfigure the CronJob to disallow new executions if the current execution is still running.

### Solution
Add the following configuration to the CronJob spec:

```yaml
concurrencyPolicy: Forbid
```

This setting prevents new executions from starting while a previous execution is still running.