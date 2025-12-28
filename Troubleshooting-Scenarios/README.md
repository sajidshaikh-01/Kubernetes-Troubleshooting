# Kubernetes Troubleshooting – Interview Style Q&A (Production Focus)
---
## 1️⃣ Pod is in `CrashLoopBackOff`. How will you debug?

### What it means

The container is **crashing repeatedly after starting**.

### Step-by-step approach

```bash
kubectl get pods
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl logs <pod-name> -p
```

### What to look for

* Application crash errors
* Wrong command or args
* Missing environment variables
* ConfigMap / Secret issues
* Port mismatch

### Interview Tip

> CrashLoopBackOff is usually an **application-level issue**, not a Kubernetes issue.

---

## 2️⃣ Pod is running but application is not accessible. What will you check?

### Debugging flow

1. Check application port inside pod

```bash
kubectl exec -it <pod> -- netstat -tuln
```

2. Check Service configuration

```bash
kubectl get svc
kubectl describe svc <service-name>
```

3. Check endpoints

```bash
kubectl get endpoints <service-name>
```

4. Verify DNS

```bash
kubectl exec -it <pod> -- nslookup <service-name>
```

### Common root causes

* Service selector mismatch
* Wrong targetPort
* Application binding to localhost instead of 0.0.0.0

---

## 3️⃣ Pod is stuck in `Pending`. Why and how do you debug?

### Meaning

Scheduler **cannot find a suitable node** for the pod.

### Debug

```bash
kubectl describe pod <pod-name>
```

### Common causes

* Insufficient CPU or memory
* NodeSelector / affinity mismatch
* PVC not bound
* Node group capacity exhausted

### Interview Line

> Pending means scheduling failure, not runtime failure.

---

## 4️⃣ Service is not routing traffic to pods. What could be wrong?

### Checklist

* Pod labels match service selectors
* Pods are Ready
* Endpoints exist

### Commands

```bash
kubectl get pods --show-labels
kubectl describe svc <service-name>
kubectl get endpoints <service-name>
```

### Interview Tip

> A Service without endpoints means traffic goes nowhere.

---

## 5️⃣ Node shows `NotReady`. How will you troubleshoot?

### Debug steps

```bash
kubectl get nodes
kubectl describe node <node-name>
```

### Check for

* DiskPressure
* MemoryPressure
* NetworkUnavailable
* Kubelet stopped

### On the node

```bash
df -h
free -m
systemctl status kubelet
```

### Production Practice

> In production, it’s often faster to replace the node.

---

## 6️⃣ ImagePullBackOff error. What are the reasons?

### Causes

* Wrong image name or tag
* Private registry authentication missing
* ImagePullSecret not configured
* DockerHub rate limit

### Debug

```bash
kubectl describe pod <pod-name>
```

---

## 7️⃣ Ingress exists but application is not accessible. How will you debug?

### Debug flow

```bash
kubectl describe ingress <ingress-name>
kubectl get pods -n ingress-nginx
kubectl logs -n ingress-nginx <controller-pod>
```

### Common causes

* Wrong service backend or port
* Ingress controller not running
* DNS not pointing to load balancer

---

## 8️⃣ Pods cannot resolve DNS names. What will you check?

### Debug

```bash
kubectl exec -it <pod> -- nslookup kubernetes.default
```

### Check CoreDNS

```bash
kubectl get pods -n kube-system
kubectl logs -n kube-system <coredns-pod>
```

### Root causes

* CoreDNS crash
* CNI networking issues
* Node network problems

---

## 9️⃣ Deployment updated but pods are not updating. Why?

### Debug

```bash
kubectl rollout status deployment <deployment-name>
kubectl describe deployment <deployment-name>
```

### Causes

* Same image tag used (latest)
* Readiness probe failing
* Rolling update paused

### Interview Line

> Image tags should be immutable in production.

---

## 🔟 Application was working yesterday, broken today. How do you approach?

### Correct mindset

> First check **what changed**.

### Checklist

* Recent deployments
* ConfigMap or Secret updates
* Image version changes
* Node scaling or failures
* Certificate expiration

### Interview Tip

This answer shows **production maturity**.

---

## 🏆 One-line Answers Interviewers Love

* CrashLoopBackOff → application crash
* Pending → scheduler cannot place pod
* NotReady node → kubelet or resource issue
* Service not working → selector or endpoints issue
* Ingress issue → controller or config problem

---

## ✅ How to Practice (Recommended)

* Break labels intentionally
* Use wrong ports
* Set low resource limits
* Stop CoreDNS
* Observe and fix using only kubectl

---

## 🧨 Real Production Kubernetes Troubleshooting Scenarios (EKS / Prod)

These scenarios are **real-world outages** commonly seen in **EKS and production Kubernetes clusters**. Interviewers expect you to explain them **calmly and step-by-step**.

---

### 🧨 Scenario 1: Pods are Running but Users Get 502 / Connection Refused

**Symptoms**

* Pods are Running
* Service exists
* Application not reachable

**Debug Steps**

```bash
kubectl get endpoints <service-name>
kubectl get pods --show-labels
kubectl describe svc <service-name>
```

**Root Cause**

* Service selector does not match pod labels

**Key Interview Line**

> A Service without endpoints means traffic goes nowhere.

---

### 🧨 Scenario 2: Pod Enters CrashLoopBackOff After New Deployment

**Symptoms**

* New release deployed
* Pod restarts continuously

**Debug Steps**

```bash
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl logs <pod-name> -p
```

**Common Root Causes**

* Missing environment variable
* ConfigMap / Secret key typo
* Wrong startup command

**Interview Insight**

> CrashLoopBackOff is usually an application issue, not Kubernetes itself.

---

### 🧨 Scenario 3: Pods Stuck in Pending During Auto Scaling

**Symptoms**

* HPA triggers
* New pods never start

**Debug Steps**

```bash
kubectl describe pod <pending-pod>
```

**Root Causes**

* Insufficient CPU / memory
* Node group max size reached
* PVC not bound

**Key Line**

> Pending means the scheduler cannot find a suitable node.

---

### 🧨 Scenario 4: Node Suddenly Becomes NotReady

**Symptoms**

* Multiple pods fail
* Node shows NotReady

**Debug Steps**

```bash
kubectl describe node <node-name>
```

**Check For**

* DiskPressure
* MemoryPressure
* NetworkUnavailable

**Production Practice**

> In production, replacing the node is often faster than fixing it.

---

### 🧨 Scenario 5: Service Works Internally but Ingress Not Accessible

**Symptoms**

* curl service works inside cluster
* Browser access fails

**Debug Steps**

```bash
kubectl describe ingress <ingress-name>
kubectl logs -n ingress-nginx <controller-pod>
```

**Root Causes**

* Wrong backend service port
* Ingress controller not running
* DNS not pointing to LoadBalancer

---

### 🧨 Scenario 6: Pods Cannot Resolve DNS Names

**Symptoms**

* Service-to-service communication fails

**Debug Steps**

```bash
kubectl exec -it <pod> -- nslookup kubernetes.default
kubectl logs -n kube-system <coredns-pod>
```

**Root Causes**

* CoreDNS crash
* CNI networking issue

---

### 🧨 Scenario 7: New Deployment Applied but Old Pods Still Serve Traffic

**Symptoms**

* CI/CD successful
* Version unchanged in production

**Debug Steps**

```bash
kubectl rollout status deployment <deployment-name>
kubectl describe deployment <deployment-name>
```

**Root Causes**

* Same image tag used (latest)
* Readiness probe failing

**Interview Line**

> Image tags should always be immutable in production.

---

### 🧨 Scenario 8: Application Restarts Randomly Under Load

**Symptoms**

* Works normally
* Crashes during peak traffic

**Debug Steps**

```bash
kubectl describe pod <pod-name>
```

**Root Cause**

* OOMKilled due to low memory limits

**Key Line**

> Requests help scheduling, limits protect the node.

---

