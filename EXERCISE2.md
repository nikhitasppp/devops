# Exercise 2: Deploy a Flask app on Minikube using kubectl and yaml

**Goal:** Build a small Flask app, containerize it, and deploy it to the local Minikube cluster, then expose it with a Service.

## Files
- [`app.py`](app.py) — minimal Flask app, responds "Hello from Flask on Kubernetes!" on port 15000
- [`Dockerfile`](Dockerfile) — builds the Flask app into a container image
- [`flask-deployment.yaml`](flask-deployment.yaml) — Kubernetes Deployment (1 replica) + NodePort Service for the app

## Steps

1. Build the image (host Docker, not Minikube's internal daemon):
   ```bash
   docker build -t flask-app .
   ```

2. Load the built image into Minikube's internal image store:
   ```bash
   minikube image load flask-app
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

```
PS> docker build -t flask-app .
PS> minikube image load flask-app
PS> kubectl apply -f flask-deployment.yaml
deployment.apps/flask-app created
service/flask-app-service created

PS> kubectl get deployments
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
flask-app   1/1     1            1           12s

PS> kubectl get pods -l app=flask-app
NAME                          READY   STATUS    RESTARTS   AGE
flask-app-59f7cdccb4-l6vf2    1/1     Running   0          21s

PS> kubectl logs flask-app-59f7cdccb4-l6vf2
 * Serving Flask app 'app'
 * Debug mode: off
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on all addresses (0.0.0.0)
 * Running on http://127.0.0.1:15000
 * Running on http://10.244.0.4:15000
Press CTRL+C to quit

PS> minikube service flask-app-service --url
http://127.0.0.1:59186
! Because you are using a Docker driver on windows, the terminal needs to be open to run it.

# in a second PowerShell window:
PS> curl http://127.0.0.1:59186
StatusCode        : 200
StatusDescription : OK
Content           : Hello from Flask on Kubernetes!
```

## Notes

- `imagePullPolicy: Never` means Kubernetes uses the local image you just built instead of trying to pull from Docker Hub — necessary here since this image only exists locally / inside Minikube's image store, not on any registry.
- `port` vs `targetPort` in the Service: `port` is what's exposed to clients (15000), `targetPort` is the container's actual listening port (also 15000 here) — the Service forwards `port` → `targetPort`.
- **Windows-specific deviation from the original exercise:** the exercise instructions use `eval $(minikube docker-env)` + `docker build` to build the image directly inside Minikube's Docker daemon. On Windows with the Docker driver, this fails with `SSH_AGENT_START: starting an SSH agent on Windows is not yet supported` — a known Minikube limitation. Worked around it with `docker build` (host Docker Desktop) + `minikube image load flask-app`, which achieves the same result (image available locally to the cluster) without needing the SSH tunnel.
- On Windows with the Docker driver, `minikube service --url` needs the terminal to stay open to keep the tunnel alive — same as Exercise 1.
