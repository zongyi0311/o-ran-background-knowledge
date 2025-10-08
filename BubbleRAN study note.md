# Setup environment
- ubuntu 24.04
- ntust server
  - IP 192.168.8.45
 
- Install BubbleRan's command Line Interface
```
ubuntu24@ubuntu24:~$ sudo snap install br-t9s --channel=beta
br-t9s (beta) v2.5.0 from bubbleran installed
```

## Have CLI already
-  check if the installation is completed
```
ubuntu24@ubuntu24:~$     cli --version
brc version v2.3.0
```

- Diagnose if the cluster is reachable(Something went wrong here.)
- **Wrong result**
```
ubuntu24@ubuntu24:~$     cli diag
2025-10-08 20:31:40 [INFO] Starting diagnostics...
2025-10-08 20:31:40 [INFO] [category:basic] [test-case:Cluster Readiness] Checking if the cluster is reachable...
E1008 20:31:42.586703   16059 memcache.go:265] "Unhandled Error" err="couldn't get current server API group list: Get \"https://192.168.8.45:6443/api?timeout=32s\": dial tcp 192.168.8.45:6443: connect: connection refused"
E1008 20:31:44.682568   16059 memcache.go:265] "Unhandled Error" err="couldn't get current server API group list: Get \"https://192.168.8.45:6443/api?timeout=32s\": dial tcp 192.168.8.45:6443: connect: connection refused"
E1008 20:31:46.763042   16059 memcache.go:265] "Unhandled Error" err="couldn't get current server API group list: Get \"https://192.168.8.45:6443/api?timeout=32s\": dial tcp 192.168.8.45:6443: connect: connection refused"
E1008 20:31:48.830625   16059 memcache.go:265] "Unhandled Error" err="couldn't get current server API group list: Get \"https://192.168.8.45:6443/api?timeout=32s\": dial tcp 192.168.8.45:6443: connect: connection refused"
E1008 20:31:50.931525   16059 memcache.go:265] "Unhandled Error" err="couldn't get current server API group list: Get \"https://192.168.8.45:6443/api?timeout=32s\": dial tcp 192.168.8.45:6443: connect: connection refused"
The connection to the server 192.168.8.45:6443 was refused - did you specify the right host or port?
```
### Summary — BubbleRAN CLI Connection Issue
- Problem Overview:
- When running cli diag, the CLI failed with a “must provide credentials” error.
Two root causes were identified:
  - Server-side: Kubernetes control-plane certificates were near expiration or not properly loaded, causing API communication issues and several pods showing Unknown status.
  - Client-side: The local kubeconfig used outdated credentials inconsistent with the renewed server certificates.
- Checked and confirmed certificate expiration on the control node.
```
sudo kubeadm certs check-expiration
```
- Regenerated all Kubernetes control-plane certificates.
```
sudo kubeadm certs renew all
```
- Restarted core services (kubelet, apiserver) to apply the new credentials.
```
sudo systemctl restart kubelet
```  
- Verified that the node returned to Ready and cluster components were functional.
```
kubectl get nodes
```
- Replaced the old local kubeconfig with the updated one from the server.
- Synced the new configuration to BubbleRAN CLI and re-ran diagnostics successfully.
```
cat .kube/config
nano ~/.kube/config ## paste the config
cp ~/.kube/config ~/snap/br-t9s/current/.kube/config
```

| Phase                     | Description                                                       | What We Did                                                                                                                                 |
| ------------------------- | ----------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| **1. Detection**          | CLI reported “must provide credentials”; Pods in `Unknown` state. | Checked kube-apiserver via `crictl`, restarted kubelet, confirmed issue persisted.                                                          |
| **2. Server-Side Repair** | Certificates expired / invalid.                                | Ran `kubeadm certs check-expiration` → renewed all certs → replaced `admin.conf` → restarted kubelet → nodes became Ready.                  |
| **3. Local Sync**         | CLI still failed authentication.                                  | Compared certs between local and server configs → replaced old kubeconfig with latest `admin.conf` → synced to Snap path → CLI diag passed. |

- **Correct result**
```
ubuntu24@ubuntu24:~$ cli diag
2025-10-08 21:29:42 [INFO] Starting diagnostics...
2025-10-08 21:29:42 [INFO] [category:basic] [test-case:Cluster Readiness] Checking if the cluster is reachable...
2025-10-08 21:29:42 [INFO] [category:basic] [test-case:Cluster Readiness] The cluster is ready and reachable.
2025-10-08 21:29:42 [INFO] [category:basic] [test-case:Cluster Readiness] Test passed
2025-10-08 21:29:42 [INFO] [category:basic] [test-case:Node Readiness] Checking if the nodes are ready...
2025-10-08 21:29:42 [INFO] [category:basic] [test-case:Node Readiness] [node:bubbleran-ee705-7] [address:192.168.8.45] Node is ready
2025-10-08 21:29:42 [INFO] [category:basic] [test-case:Node Readiness] All nodes are ready.
2025-10-08 21:29:42 [INFO] [category:basic] [test-case:Node Readiness] Test passed
2025-10-08 21:29:42 [INFO] [category:basic] [test-case:Node Resources] Checking if the nodes have available resources...
2025-10-08 21:29:42 [INFO] [category:basic] [test-case:Node Resources] [node:bubbleran-ee705-7] [resource:gaia.t9s.io/net-tun] Node has available resources
2025-10-08 21:29:42 [INFO] [category:basic] [test-case:Node Resources] [node:bubbleran-ee705-7] [resource:memory] Node has available resources
2025-10-08 21:29:42 [INFO] [category:basic] [test-case:Node Resources] [node:bubbleran-ee705-7] [resource:pods] Node has available resources
2025-10-08 21:29:42 [INFO] [category:basic] [test-case:Node Resources] [node:bubbleran-ee705-7] [resource:cpu] Node has available resources
2025-10-08 21:29:42 [INFO] [category:basic] [test-case:Node Resources] [node:bubbleran-ee705-7] [resource:ephemeral-storage] Node has available resources
2025-10-08 21:29:42 [INFO] [category:basic] [test-case:Node Resources] Test passed
2025-10-08 21:29:42 [INFO] [category:basic] [test-case:Operator Installation] Checking if the operators are ready...
2025-10-08 21:29:43 [INFO] [category:basic] [test-case:Operator Installation] Gaia configuration file not found. Skipping operator check.
2025-10-08 21:29:43 [INFO] [category:basic] [test-case:Operator Installation] Test passed
2025-10-08 21:29:43 [INFO] Diagnostics completed successfully.
```
