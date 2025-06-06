# Exercise 5: Blue-Green Deployment

This directory contains the necessary files for implementing the blue-green deployment exercise described in `exercise_deployments.md`.

## Files Created

1. **nginx-blue-deployment.yaml**: Deployment for the initial (blue) version
   - 3 replicas
   - nginx:1.23.0 container image
   - Label: version=blue

2. **nginx-service.yaml**: Service to expose the deployment
   - Type: ClusterIP
   - Port: 80
   - Initially selects pods with label version=blue

3. **nginx-green-deployment.yaml**: Deployment for the new (green) version
   - 3 replicas
   - nginx:1.23.4 container image
   - Label: version=green

4. **blue-green-switch-commands.md**: Step-by-step instructions for:
   - Creating the initial blue deployment
   - Testing with a temporary curl pod
   - Deploying the green version
   - Switching the service from blue to green
   - Testing the green deployment
   - Cleaning up the blue deployment

## Exercise Flow

1. Start with the blue deployment (nginx:1.23.0)
2. Expose it with a service
3. Test the service
4. Deploy the green version (nginx:1.23.4)
5. Switch the service to point to green pods
6. Test the service again
7. Remove the blue deployment

This demonstrates a zero-downtime deployment strategy where a new version is fully deployed before any traffic is switched over.