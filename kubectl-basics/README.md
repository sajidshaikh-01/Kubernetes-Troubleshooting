# Kubernetes Troubleshooting Commands – Meaning & Examples
---

## 1️⃣ kubectl get (k get)

### Purpose

Used to **list Kubernetes resources**.

### Common Usage

```bash
kubectl get pods
kubectl get svc
kubectl get nodes
kubectl get deployments
```

### Example

```bash
kubectl get pods -o wide
```

➡ Shows pod name, status, node, IP, etc.

📌 **Used first in almost every troubleshooting case**.

---

## 2️⃣ kubectl get events (k get events)

### Purpose

Shows **what is happening inside the cluster** (failures, scheduling issues, restarts).

### Command

```bash
kubectl get events
```

### Sorted by Time (Very Useful)

```bash
kubectl get events --sort-by=.metadata.creationTimestamp
```

### Example Use Case

* Pod stuck in `Pending`
* ImagePullBackOff
* CrashLoopBackOff

📌 **Events often tell the real root cause**.

---

## 3️⃣ kubectl logs (k logs)

### Purpose

Fetch logs from a container.

### Command

```bash
kubectl logs <pod-name>
```

### Multi-container Pod

```bash
kubectl logs <pod-name> -c <container-name>
```

📌 Used when application is crashing or misbehaving.

---

## 4️⃣ kubectl logs --selector / --label (k logs -l)

### Purpose

Fetch logs from **multiple pods at once** using labels.

### Command

```bash
kubectl logs -l app=nginx
```

### Example

```bash
kubectl logs -l app=backend --all-containers=true
```

📌 **Very useful for debugging deployments & microservices**.

---

## 5️⃣ kubectl logs --timestamps

### Purpose

Adds timestamps to log entries (important for production debugging).

### Command

```bash
kubectl logs <pod> --timestamps
```

### Example

Helps correlate logs with:

* Alerts
* Prometheus metrics
* Incident timelines

📌 **Highly recommended in production**.

---

## 6️⃣ kubectl logs --follow (k logs -f)

### Purpose

Stream logs in real time (like `tail -f`).

### Command

```bash
kubectl logs -f <pod>
```

### Example Use Case

* Watching startup logs
* Debugging crashes live

📌 Commonly used during deployments.

---

## 7️⃣ kubectl exec

### Purpose

Execute commands **inside a running container**.

### Command

```bash
kubectl exec -it <pod> -- sh
```

### Example

```bash
kubectl exec -it <pod> -- curl localhost:8080
```

### Use Cases

* Check app health
* Inspect files
* Test network connectivity

📌 **Must-have skill for live debugging**.

---

## 8️⃣ kubectl port-forward (k port-forward)

### Purpose

Forward a local port to a pod or service.

### Command

```bash
kubectl port-forward pod/<pod-name> 8080:80
```

### Service Example

```bash
kubectl port-forward svc/backend 8080:80
```

### Use Cases

* Access app locally without Ingress
* Debug internal services

📌 Very useful in dev & troubleshooting.

---

## 9️⃣ kubectl auth can-i

### Purpose

Check **RBAC permissions**.

### Command

```bash
kubectl auth can-i get pods
```

### With User / ServiceAccount

```bash
kubectl auth can-i create pods --as system:serviceaccount:default:my-sa
```

### Example Use Case

* Permission denied errors
* Debugging RBAC issues

📌 **Frequently used by platform engineers**.

---

## 🔟 kubectl top

### Purpose

Show **CPU & memory usage**.

### Command

```bash
kubectl top nodes
kubectl top pods
```

### Example

```bash
kubectl top pods --all-namespaces
```

📌 Requires **metrics-server** installed.

Used to:

* Identify resource-hungry pods
* Debug OOMKilled issues

---

## 1️⃣1️⃣ kubectl explain

### Purpose

Shows **schema & documentation** of Kubernetes resources.

### Command

```bash
kubectl explain pod
```

### Deep Dive Example

```bash
kubectl explain pod.spec.containers.resources
```

📌 Very helpful during interviews & YAML writing.

---

## 1️⃣2️⃣ kubectl diff

### Purpose

Show differences **before applying changes**.

### Command

```bash
kubectl diff -f deployment.yaml
```

### Example Use Case

* GitOps
* CI/CD validation
* Prevent breaking changes

📌 Safer alternative to blind `kubectl apply`.

---
