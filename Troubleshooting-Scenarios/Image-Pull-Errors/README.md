# Kubernetes Image Pull Errors – Causes & Solutions (Production Guide)
---
## ❌ Common Image Pull Error States

* `ErrImagePull`
* `ImagePullBackOff`

### Meaning

Kubernetes **failed to download the container image** from the registry.

---

## 🔍 How to Identify Image Pull Errors

```bash
kubectl get pods
kubectl describe pod <pod-name>
```

Check the **Events** section for messages like:

* `Failed to pull image`
* `pull access denied`
* `manifest unknown`
* `no basic auth credentials`

---

## 1️⃣ Wrong Image Name or Tag

### Cause

* Typo in image name
* Image tag does not exist
* Image pushed to a different registry

### Example Error

```text
manifest unknown: manifest unknown
```

### Solution

* Verify image name and tag
* Manually pull the image to confirm

```bash
docker pull nginx:1.25
```

📌 **Production Tip:** Avoid using `latest` tag.

---

## 2️⃣ Private Registry Authentication Missing

### Cause

* Image is private
* No ImagePullSecret configured

### Example Error

```text
pull access denied for repo, repository does not exist or may require authorization
```

### Solution

Create Docker registry secret:

```bash
kubectl create secret docker-registry regcred \
  --docker-server=https://index.docker.io/v1/ \
  --docker-username=<username> \
  --docker-password=<password> \
  --docker-email=<email>
```

Attach secret to pod/deployment:

```yaml
imagePullSecrets:
  - name: regcred
```

---

## 3️⃣ ImagePullSecret Not Attached to ServiceAccount

### Cause

* Secret exists but not linked

### Solution

Attach secret to ServiceAccount:

```bash
kubectl patch serviceaccount default \
  -p '{"imagePullSecrets": [{"name": "regcred"}]}'
```

📌 Ensures all pods using this ServiceAccount can pull images.

---

## 4️⃣ DockerHub Rate Limit Exceeded

### Cause

* Too many anonymous pulls

### Symptoms

* Image pulls fail intermittently

### Solution

* Authenticate DockerHub
* Use private registry (ECR, ACR, GCR)

📌 Very common in CI/CD pipelines.

---

## 5️⃣ Wrong Registry or Repository Path

### Cause

* Image pushed to ECR/ACR/GCR
* Deployment points to wrong registry

### Example

```yaml
image: nginx:latest   # Wrong
image: 123456789012.dkr.ecr.ap-south-1.amazonaws.com/nginx:1.0
```

### Solution

* Confirm correct registry URL
* Validate region and account ID

---

## 6️⃣ Node Has No Internet / Network Issue

### Cause

* Private nodes without NAT Gateway
* Firewall / proxy blocking registry

### Debug

SSH into node and test:

```bash
curl https://registry-1.docker.io
```

### Solution

* Configure NAT Gateway
* Allow outbound internet access

📌 Common in private EKS clusters.

---

## 7️⃣ Image Architecture Mismatch

### Cause

* ARM image on AMD64 node (or vice versa)

### Example Error

```text
exec format error
```

### Solution

* Build multi-arch images
* Match node architecture

```bash
docker buildx build --platform linux/amd64,linux/arm64 .
```

