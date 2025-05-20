## 📘 Scenario #1: Zombie Pods Causing `kubectl drain` to Hang

- **Category**: Cluster Management  
- **Environment**: Kubernetes v1.23, On-prem Bare Metal, Systemd cgroups  

### 🧩 Scenario Summary
`kubectl drain` stuck indefinitely due to an unresponsive terminating pod.

### 🔍 What Happened
A pod with a custom finalizer never completed termination, which blocked the `kubectl drain` command. Although the pod was marked for deletion, the API server continued waiting because the finalizer was not removed.

### 🧪 Diagnosis Steps
- Ran `kubectl get pods --all-namespaces -o wide` to identify lingering pods.
- Found a pod stuck in the `Terminating` state for over 20 minutes.
- Used `kubectl describe pod <pod>` and discovered a custom finalizer.
- Checked logs of the controller managing the finalizer — the controller had crashed.

### 🚨 Root Cause
Finalizer logic was not executed because the controller responsible for it was down, leaving the pod in an undeletable state.

### 🛠️ Fix / Workaround
```bash
kubectl patch pod <pod-name> -p '{"metadata":{"finalizers":[]}}' --type=merge
