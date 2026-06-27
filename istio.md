# 🧩 Problem Statement(Istio mTLS Enforcement- Layer 4 Security)

<details>
<summary>Click to expand the problem statement</summary>


A microservices application is running in the Kubernetes cluster.
Currently, service-to-service communication within the application uses unencrypted **Layer 4 (TCP)** traffic.

Istio has already been installed in the cluster to help secure internal communication using **mutual TLS (mTLS)**.

The workloads are deployed inside the `istio-example` namespace, but secure communication is not yet fully enforced.

---

### 🎯 Task

To secure all Layer 4 traffic, complete the following:

1. Ensure every Pod running in the `istio-example` namespace has the **Istio sidecar proxy (`istio-proxy`) injected**.
2. Configure **mutual TLS authentication in STRICT mode** for all workloads inside the `istio-example` namespace.

---

### 📌 Notes

* Istio is already installed and available.
* You are allowed to refer to the official Istio documentation during the exam.
* After configuration, all internal service communication must use encrypted mTLS only.

</details>

# ✅ Solution

<details>
<summary>Click to expand the solution</summary>


In the CKS exam, you are allowed to access the official Istio documentation.
At the beginning of the question, a documentation link is usually provided — make sure to use it.

Reference:
[https://istio.io/latest/docs/reference/config/security/peer_authentication/](https://istio.io/latest/docs/reference/config/security/peer_authentication/)

---

### Step 1: Verify existing Pods

Check how many Pods are running inside the namespace:

```
kubectl get pods -n istio-example
```

---

### Step 2: Enable Istio Sidecar Injection

To inject the `istio-proxy` sidecar into all Pods in the namespace:

```
kubectl label namespace istio-example istio-injection=enabled
```

Verify label:

```
kubectl get ns istio-example --show-labels
```

---

### Step 3: Restart Deployments

Existing Pods will not automatically get sidecars.
You must restart the deployments:

```
kubectl rollout restart deployment -n istio-example
```

Verify sidecar injection:

```
kubectl get pods -n istio-example
```

You should now see **2 containers per Pod** (application + istio-proxy).

---

### Step 4: Enforce mTLS in STRICT Mode

Create a `PeerAuthentication` resource:

```yaml
apiVersion: security.istio.io/v1
kind: PeerAuthentication
metadata:
  name: default
  namespace: istio-example
spec:
  mtls:
    mode: STRICT
```

Apply it:

```
kubectl apply -f peerauth.yaml
```

Now all workloads in the namespace will only accept encrypted mTLS traffic.

</details>