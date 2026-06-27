# 🧩 Problem Statement(Container Security Context Hardening)

<details>
<summary>Click to expand the problem statement</summary>

A Kubernetes Deployment is running with insecure container settings.

Your objective is to harden the containers by enforcing immutability and reducing privilege-related risks.

An existing Deployment named **`secdep`** is deployed in the **`sec-ns`** namespace.

You have been provided with the manifest file:

```
~/sec-ns_deployment.yaml
```

---

### 🎯 Task

Update the Deployment so that all containers:

1. Run as a non-root user with **UID 32000**
2. Use a **read-only root filesystem**
3. **Disallow privilege escalation**

---

### 📌 Notes

* Modify the Deployment manifest file.
* Re-apply the updated manifest.
* Focus only on container-level security context settings required by the task.

</details>

# ✅ Solution

<details>
<summary>Click to expand the solution</summary>

### Step 1: Open the Deployment manifest

```
vim ~/sec-ns_deployment.yaml
```

---

### Step 2: Add securityContext at the container level

Inside the container specification, add:

```yaml
securityContext:
  runAsUser: 32000
  allowPrivilegeEscalation: false
  readOnlyRootFilesystem: true
```

Example:

```yaml
spec:
  containers:
  - name: nginx
    image: nginx:1.27
    ports:
    - containerPort: 80
    securityContext:
      runAsUser: 32000
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
```

⚠️ Important:

* It must be `securityContext` (lowercase `s`)
* It must be inside the container block

---

### Step 3: Apply the updated manifest

```
kubectl apply -f ~/sec-ns_deployment.yaml
```

---

### Step 4: Verify Pods

```
kubectl get pods -n sec-ns
```

Ensure the Pod restarts successfully with the updated configuration.

</details>