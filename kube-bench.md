# 🧩 Problem Statement(Hardening Kubelet and ETCD Configuration)

<details>
<summary>Click to expand the problem statement</summary>

A Kubernetes cluster created using **kubeadm** was scanned using a **CIS Benchmark tool**, and multiple critical security violations were reported.

Your task is to:

1. Log in to the **appropriate node**
2. Identify and fix the reported security issues
3. Restart the affected components so the new configuration takes effect

### 🔐 Fix the following violations related to kubelet:

* Ensure `anonymous-auth` is set to `false`
* Ensure `--authorization-mode` is **not** set to `AlwaysAllow`
  
`Note`: Wherever possible, use **Webhook-based authentication and authorization**

### 🔐 Fix the following violation related to etcd:

* Ensure the `--client-cert-auth` parameter is set to `true`

</details>

# ✅ Solution

<details>
<summary>Click to expand the solution</summary>


### Step 1: Locate the kubelet configuration file

By default, it is located at:

```
/var/lib/kubelet/config.yaml
```

If unsure, verify using:

```
ps -ef | grep kubelet
```

Look for the `--config` argument to confirm the file path.

---

### Step 2: Update kubelet configuration

Open the file:

```
sudo vim /var/lib/kubelet/config.yaml
```

Modify the configuration as shown below:

```yaml
authentication:
  anonymous:
    enabled: false   # Change from true to false
  webhook:
    enabled: true    # Change from false to true
    cacheTTL: 0s
  x509:
    clientCAFile: /etc/kubernetes/pki/ca.crt

authorization:
  mode: Webhook      # Change from AlwaysAllow to Webhook
  webhook:
    cacheAuthorizedTTL: 0s
    cacheUnauthorizedTTL: 0s
```

Save and exit.

---

### Step 3: Update ETCD configuration

Open the etcd static pod manifest:

```
sudo vim /etc/kubernetes/manifests/etcd.yaml
```

Ensure the following flag is set:

```yaml
- --client-cert-auth=true   # Change from false to true
```

Save and exit.

---

### Step 4: Restart kubelet

```
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

---

### Step 5: Verify cluster status

```
kubectl get nodes
```

If the node shows `Ready` status and the API server is reachable, the configuration has been applied successfully.

</details>