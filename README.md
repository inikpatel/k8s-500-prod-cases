## 📘 Scenario #1: Zombie Pods Causing NodeDrain to Hang

- **Category**: Cluster Management  
- **Environment**: K8s v1.23, On-prem bare metal, Systemd cgroups  

### Scenario Summary
Node drain stuck indefinitely due to unresponsive terminating pod.

### What Happened
A pod with a custom finalizer never completed termination, blocking `kubectl drain`. Even after the pod was marked for deletion, the API server kept waiting because the finalizer wasn’t removed.

### Diagnosis Steps
- Checked `kubectl get pods --all-namespaces -o wide` to find lingering pods.
- Found pod stuck in `Terminating` state for over 20 minutes.
- Used `kubectl describe pod <pod>` to identify the presence of a custom finalizer.
- Investigated controller logs managing the finalizer – the controller had crashed.

### Root Cause
Finalizer logic was never executed because its controller was down, leaving the pod undeletable.

### Fix/Workaround
```bash
kubectl patch pod <pod-name> -p '{"metadata":{"finalizers":[]}}' --type=merge

📘 Scenario #1: Zombie Pods Causing NodeDrain to Hang  
Category: Cluster Management  
Environment: K8s v1.23, On-prem bare metal, Systemd cgroups  
Scenario Summary: Node drain stuck indefinitely due to unresponsive terminating pod.  
What Happened: A pod with a custom finalizer never completed termination, blocking kubectl drain. Even after the pod was marked for deletion, the API server kept waiting because the finalizer wasn't removed.  
Diagnosis Steps:  
	• Checked kubectl get pods --all-namespaces -o wide to find lingering pods.  
	• Found pod stuck in Terminating state for over 20 minutes.  
	• Used kubectl describe pod <pod> to identify the presence of a custom finalizer.  
	• Investigated controller logs managing the finalizer – the controller had crashed.  
Root Cause: Finalizer logic was never executed because its controller was down, leaving the pod undeletable.  
Fix/Workaround:  
kubectl patch pod <pod-name> -p '{"metadata":{"finalizers":[]}}' --type=merge  
Lessons Learned: Finalizers should have timeout or fail-safe logic.  
How to Avoid:  
	• Avoid finalizers unless absolutely necessary.  
	• Add monitoring for stuck Terminating pods.  
	• Implement retry/timeout logic in finalizer controllers.  

📘 Scenario #2: API Server Crash Due to Excessive CRD Writes  
Category: Cluster Management  
Environment: K8s v1.24, GKE, heavy use of custom controllers  
Scenario Summary: API server crashed due to flooding by a malfunctioning controller creating too many custom resources.  
What Happened: A bug in a controller created thousands of Custom Resources (CRs) in a tight reconciliation loop. Etcd was flooded, leading to slow writes, and the API server eventually became non-responsive.  
Diagnosis Steps:  
	• API latency increased, leading to 504 Gateway Timeout errors in kubectl.  
	• Used kubectl get crds | wc -l to list all CRs.  
	• Analyzed controller logs – found infinite reconcile on a specific CR type.  
	• etcd disk I/O was maxed.  
Root Cause: Bad logic in reconcile loop: create was always called regardless of the state, creating resource floods.  
Fix/Workaround:  
	• Scaled the controller to 0 replicas.  
	• Manually deleted thousands of stale CRs using batch deletion.  
Lessons Learned: Always test reconcile logic in a sandboxed cluster.  
How to Avoid:  
	• Implement create/update guards in reconciliation.  
	• Add Prometheus alert for high CR count.  

[... rest of scenarios follow the same pattern ...]
