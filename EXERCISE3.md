# Exercise 3: Scaling a Flask App with ReplicaSets

**Scenario:** An e-commerce flash sale app needs to handle a sudden spike in traffic. A single Pod would crash under load — a ReplicaSet keeps several identical copies running and automatically replaces any that die.

## Files
- [`exercise3/app.py`](exercise3/app.py) — Flask "flash sale" app with `/` (welcome), `/buy` (simulated checkout, shows which Pod served it), `/health` (for probes)
- [`exercise3/Dockerfile`](exercise3/Dockerfile) — builds the app with gunicorn
- [`exercise3/flashsale-replicaset.yaml`](exercise3/flashsale-replicaset.yaml) — a ReplicaSet (3 replicas) + ClusterIP Service

## Steps

1. (Optional clean slate) Reset Minikube to a single node:
   ```bash
   minikube stop
   minikube delete
   minikube start --nodes=1 --driver=docker
   ```

2. Build and load the image (same pattern as Exercise 2 — host Docker build, then load into Minikube):
   ```bash
   cd exercise3
   docker build -t flashsale:1.0 .
   minikube image load flashsale:1.0
   ```

3. Apply the ReplicaSet + Service:
   ```bash
   kubectl apply -f flashsale-replicaset.yaml
   ```

4. Verify the 3 pods and the ReplicaSet:
   ```bash
   kubectl get pods
   kubectl get rs
   ```

5. Scale up to 5 replicas:
   ```bash
   kubectl scale rs flashsale-rs --replicas=5
   kubectl get rs
   kubectl get pods
   ```

6. Kill one pod and watch Kubernetes replace it automatically:
   ```bash
   kubectl delete pod <one-of-the-pod-names-from-above>
   kubectl get pods
   ```

7. See pod distribution (all on the one node, since we're running `--nodes=1`):
   ```bash
   kubectl get pods -o wide
   ```

## Result

_(to fill in: actual output of each command above)_

## Notes

- Added `imagePullPolicy: Never` to the ReplicaSet spec (not in the original exercise text) — needed because the image only exists locally via `minikube image load`, same reasoning as Exercise 2.
- `ReplicaSet` vs `Deployment`: a ReplicaSet just keeps N identical Pods running. A Deployment manages ReplicaSets for you and adds rolling updates/rollback — in practice you almost always use a Deployment, but this exercise uses a bare ReplicaSet directly so the scaling/self-healing mechanics are visible without a Deployment's extra layer in the way.

## Q&A

**Q1. What is the initial number of replicas in the ReplicaSet?**
A: 3 (set in `spec.replicas` in `flashsale-replicaset.yaml`)

**Q2. How many pods are running after applying the ReplicaSet configuration?**
A: _(fill in from `kubectl get pods` output)_

**Q3. What happens when you scale the ReplicaSet to 5 replicas?**
A: Kubernetes creates 2 additional pods to reach the desired count of 5.

**Q4. What happens when you delete one pod?**
A: The ReplicaSet controller notices the pod count dropped below the desired replica count and creates a new pod to replace it — this is the self-healing behavior.

**Q5. How does Kubernetes maintain the desired number of replicas?**
A: The ReplicaSet controller continuously compares the actual pod count against `spec.replicas` and creates or deletes pods as needed to match.

**Q6. How many nodes are running?**
A: 1 (started with `minikube start --nodes=1`)

**Q7. Where are the pods running with respect to nodes?**
A: _(fill in from `kubectl get pods -o wide` — with a single node, all pods run on that one `minikube` node)_
