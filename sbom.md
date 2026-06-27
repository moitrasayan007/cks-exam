# 🧩 Problem Statement(BOM Tool)

There is a Deployment named **`sbom`** running in the **`sbom`** namespace.

This Deployment contains **three containers**, and each container uses a **different image version**.

Your task is to inspect the container images and identify **which container includes** the package:

* `libcrypto3` **version** `3.1.4-r5`

After identifying the correct image version:

1. Use the pre-installed **`bom`** utility to generate an **SPDX (Software Bill of Materials)** report and save it as:

   ```
   ~/report.spdx
   ```
2. Update the **`sbom` Deployment** so that the container using the identified image version is **removed**.

The Deployment manifest file is located at:

```
~/sbom-deployment.yaml
```

---

### ✅ Important Constraints

* Do **not** modify the other containers.
* Only remove the container that uses the identified (Alpine-based) image version.


# ✅ Solution

### Step 1: Identify the Pod name

Get the Pod running in the `sbom` namespace:

```
kubectl get pods -n sbom
```

Assume the Pod name is:

```
<POD_NAME>
```

---

### Step 2: Check which container has `libcrypto3` version `3.1.4-r5`

Run the following for each container:

```
kubectl exec -n sbom <POD_NAME> -c container-1 -- apk list --installed | grep libcrypto3
kubectl exec -n sbom <POD_NAME> -c container-2 -- apk list --installed | grep libcrypto3
kubectl exec -n sbom <POD_NAME> -c container-3 -- apk list --installed | grep libcrypto3
```

Example output:

```
container-1:
libcrypto3-3.3.0-r2 ...

container-2:
libcrypto3-3.1.4-r5 ...

container-3:
(no output)
```

So, **container-2** contains the target package version.

---

### Step 3: Generate SBOM (SPDX) using bom

Now generate the SPDX report for the identified image:

```
bom generate --image devopstechtales/cks-exam-questions:sbom-test-v2 -o ~/report.spdx
```

Verify the file exists:

```
ls -l ~/report.spdx
```

---

### Step 4: Remove the affected container from the Deployment

Edit the Deployment manifest:

```
vim ~/sbom-deployment.yaml
```

Remove only this container block:

```yaml
- name: container-2
  image: devopstechtales/cks-exam-questions:sbom-test-v2
  command: ["sleep", "3600"]
```

Save and apply the updated manifest:

```
kubectl apply -f ~/sbom-deployment.yaml
```

---

### Step 5: Verify the Deployment

Check Pods again:

```
kubectl get pods -n sbom
```

Optionally confirm the container list in the new Pod:

```
kubectl get pod -n sbom <NEW_POD_NAME> -o jsonpath='{.spec.containers[*].name}'; echo
```

You should now see only:

```
container-1 container-3
```