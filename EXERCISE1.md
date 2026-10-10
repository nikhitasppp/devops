# Exercise 1: Hello Pod

**Goal:** Run your first app inside Kubernetes and access it.

## Steps

1. Start a local Kubernetes cluster:
   ```bash
   minikube start
   ```

2. Create the first Pod (Nginx image):
   ```bash
   kubectl run hello-k8s --image=nginx --port=80
   ```

3. Verify the Pod is running:
   ```bash
   kubectl get pods
   ```

4. Expose the Pod as a Service:
   ```bash
   kubectl expose pod hello-k8s --type=NodePort --port=80
   ```

5. Open the app in the browser:
   ```bash
   minikube service hello-k8s
   ```

## Result

_(to fill in: output of `kubectl get pods`, `kubectl get svc`, and a screenshot of the Nginx welcome page)_

## Notes

_(to fill in: what each command does, and anything that went wrong / how it was fixed)_
