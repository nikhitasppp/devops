# Exercise 2: Deploy a Flask app on Minikube using kubectl and yaml

**Goal:** Build a small Flask app, containerize it, and deploy it to the local Minikube cluster, then expose it with a Service.

## Files
- [`app.py`](app.py) — minimal Flask app, responds "Hello from Flask on Kubernetes!" on port 15000
- [`Dockerfile`](Dockerfile) — builds the Flask app into a container image
- [`flask-deployment.yaml`](flask-deployment.yaml) — Kubernetes Deployment (1 replica) + NodePort Service for the app

## Steps

1. Point the local Docker CLI at Minikube's internal Docker daemon (so the image is built *inside* Minikube, not on the host):
   ```bash
   eval $(minikube docker-env)
   ```
   (On Windows PowerShell: `& minikube docker-env | Invoke-Expression`)

2. Build the image:
   ```bash
   docker build -t flask-app .
   ```

3. Deploy (creates both the Deployment and the Service, since both are in `flask-deployment.yaml`):
   ```bash
   kubectl apply -f flask-deployment.yaml
   ```

4. Check the deployment and pod:
   ```bash
   kubectl get deployments
   kubectl get pods -l app=flask-app
   ```

5. Check logs (confirm Flask actually started):
   ```bash
   kubectl logs <pod-name>
   ```

6. Get the reachable URL and test it:
   ```bash
   minikube service flask-app-service --url
   # in a NEW terminal, with the first one still running:
   curl http://127.0.0.1:<port-from-above>
   ```

## Result

_(to fill in: output of each command above, plus the curl response)_

## Notes

_(to fill in: `imagePullPolicy: Never` means Kubernetes uses the local image you just built instead of trying to pull from Docker Hub — important since this image only exists inside Minikube's Docker daemon. `port` vs `targetPort` in the Service: `port` is what's exposed to clients, `targetPort` is the container's actual listening port — here they're both 15000.)_
