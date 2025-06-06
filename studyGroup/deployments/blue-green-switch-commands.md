# Blue-Green Deployment Switch Commands

## Initial Setup
```bash
# Create the blue deployment
kubectl apply -f nginx-blue-deployment.yaml

# Create the service pointing to blue deployment
kubectl apply -f nginx-service.yaml

# Verify the service is pointing to blue pods
kubectl describe service nginx
```

## Test Blue Deployment
```bash
# Run a temporary pod to test the service
kubectl run curl --image=alpine/curl:3.14 --rm -it -- sh -c 'curl nginx'
```

## Deploy Green Version
```bash
# Create the green deployment
kubectl apply -f nginx-green-deployment.yaml
```

## Switch Traffic from Blue to Green
```bash
# Update the service to point to green deployment
kubectl patch service nginx -p '{"spec":{"selector":{"version":"green"}}}'

# Verify the service is now pointing to green pods
kubectl describe service nginx
```

## Test Green Deployment
```bash
# Run a temporary pod to test the service
kubectl run curl --image=alpine/curl:3.14 --rm -it -- sh -c 'curl nginx'
```

## Cleanup Blue Deployment
```bash
# Delete the blue deployment when green is confirmed working
kubectl delete -f nginx-blue-deployment.yaml
```