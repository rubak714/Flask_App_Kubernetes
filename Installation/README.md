## Installing and Using `kubectl` with Minikube

### Option 1: Install `kubectl` Globally (Recommended)
This lets you use `kubectl` across all Kubernetes clusters — not just Minikube.

#### Steps:
```bash
curl -LO https://dl.k8s.io/release/v1.33.1/bin/linux/amd64/kubectl
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```

#### Verify Installation:
```bash
kubectl version --client
```

---

### Option 2: Use `kubectl` via Minikube (No Global Install)
If you don't want to install globally, Minikube bundles its own `kubectl`.

#### Example:
```bash
minikube kubectl -- get pods -A
```
This runs `kubectl` only within the Minikube environment.

You can also alias it:
```bash
alias kubectl='minikube kubectl --'
```

---

## Useful `kubectl` Commands

### 1. View all running Pods in all namespaces:
```bash
kubectl get pods -A
```

#### Output Explanation:
| NAMESPACE     | POD NAME                          | PURPOSE / COMPONENT                                 |
|---------------|-----------------------------------|------------------------------------------------------|
| kube-system   | coredns-*                         | DNS for service discovery inside cluster             |
| kube-system   | etcd-minikube                     | Stores entire cluster state                         |
| kube-system   | kube-apiserver-minikube           | Accepts API requests (`kubectl`, dashboard, etc.)   |
| kube-system   | kube-controller-manager-minikube  | Maintains desired cluster state                     |
| kube-system   | kube-scheduler-minikube           | Assigns Pods to Nodes                               |
| kube-system   | kube-proxy-*                      | Handles Pod networking                              |
| kube-system   | storage-provisioner               | Auto-creates volumes when requested                 |

---

### 2. View your cluster’s nodes:
```bash
kubectl get nodes
```
Example output:
| NODE NAME | STATUS | ROLES         | VERSION  |
|-----------|--------|---------------|----------|
| minikube  | Ready  | control-plane | v1.33.1  |

---

### 3. Check the current context:
```bash
kubectl config current-context
```
Expected output:
```
minikube
```
This confirms that `kubectl` is currently pointing to your local Minikube cluster.

---

Let me know if you want to deploy your first app or set up the Minikube dashboard!
