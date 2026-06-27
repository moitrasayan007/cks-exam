# 🧩 Problem Statement(Detecting a Pod Accessing `/dev/mem`)


A suspicious Pod in the cluster has been flagged for abnormal behavior.

A Pod belonging to the **neuron** application is attempting to access the sensitive host device:

```
/dev/mem
```

This indicates a potential security threat and must be stopped.

---

### 🎯 Task

1. Identify the Pod that is accessing `/dev/mem`.
2. Scale the Deployment of the identified Pod down to **zero replicas**.


# ✅ Solution


<summary>Click to expand the solution</summary>

### Step 1: Check Pods in the `neuron` namespace

```
kubectl get pods -n neuron
```

---

### Step 1: Create a Falco rule to detect `/dev/mem` access

Edit Falco local rules file:

```
sudo vim /etc/falco/falco_rules.local.yaml
```

Add the following rule:

```yaml
- list: sensitive_devices
  items: [/dev/mem]

- rule: Container Accesses /dev/mem
  desc: Detect containers attempting to read or write /dev/mem
  condition: >
    evt.type in (open, openat, openat2) and fd.name in (sensitive_devices)
  output: >
    container_id=%container.id 
  priority: CRITICAL
```

Save and exit.

---

### Step 2: Run Falco using the local rules

Run Falco in the foreground so you can immediately see events:

```
sudo falco -r /etc/falco/falco_rules.local.yaml
```

When the suspicious access happens, Falco will print an alert including:

* `container_id`
* Kubernetes namespace
* Pod name (often available as `k8s.pod`)
* Container name

Example (illustration):

```
Suspicious /dev/mem access ... container_id=abcd1234 k8s.ns=neuron k8s.pod=neuron-xxx
```

---

### Step 3: Map container ID to Pod/Deployment (if needed)

If Falco output includes the Pod name already, you can skip this step.

Otherwise, use CRI tooling to identify the container:

```
sudo crictl ps | grep <container-id>
```

Then find the Pod and Deployment via Kubernetes labels:

```
kubectl get pod -n neuron -o wide
kubectl describe pod -n neuron <pod-name>
```

---

### Step 4: Scale the Deployment down to 0 replicas

Once you identify the Deployment, scale it down:

```
kubectl scale deployment -n neuron <deployment-name> --replicas=0
```

Verify:

```
kubectl get deploy,pods -n neuron
```

Pods for that Deployment should terminate.