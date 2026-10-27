# Lab 10: Node Isolation Using Taints and Labels in Kubernetes

## Objective
Apply a taint to a worker node so pods cannot be scheduled on it without a matching toleration, label the same node for targeted scheduling, and verify both with `kubectl describe node` and `kubectl get nodes --show-labels`.

## Environment
- Cluster from Lab 9: `master-1` (control plane) and `worker-1`, Kubernetes `v1.28.15`
- Commands run on `master-1` over SSH from PowerShell on Windows 10

## Steps

### 1. Check the cluster
```bash
kubectl get pods -A
kubectl get nodes
```
Both nodes are `Ready` and the system pods are `Running`.

### 2. Taint the worker node
```bash
kubectl taint nodes worker-1 workload=ivolveApp:NoSchedule
```
- key: `workload`, value: `ivolveApp`, effect: `NoSchedule`
- New pods without a toleration for this taint will not be scheduled on `worker-1`.

### 3. Label the worker node
```bash
kubectl label nodes worker-1 project=ivolve
```

### 4. Verify
```bash
kubectl describe node worker-1 | grep -i taint
kubectl get nodes --show-labels
```
- `Taints: workload=ivolveApp:NoSchedule`
- The `worker-1` labels include `project=ivolve`

![Taint and label](screenshots/lab10-node-taint-label.png)

### 5. Remove the taint (cleanup)
```bash
kubectl taint nodes worker-1 workload=ivolveApp:NoSchedule-
kubectl get nodes --show-labels
```
The label `project=ivolve` stays on the node.

## Notes and issues
- **Why I removed the taint:** the cluster has one worker, and the master already has the control-plane taint. Leaving `NoSchedule` on `worker-1` would keep pods in Labs 11 and 12 `Pending`, so I removed the taint after taking the screenshot. The label was kept.
- **SSH to the master timed out:** before this lab, `ssh k8s@192.168.247.130` failed with `Connection timed out`. I installed and checked the SSH server (`sudo apt update`, `sudo apt install openssh-server`, then confirmed the service was enabled and running), confirmed both VM IPs were unchanged, and ran `Test-NetConnection 192.168.247.130 -Port 22` from PowerShell, which kept failing. Restarting `master-1` and `worker-1` fixed it, and I did not identify a deeper cause.
- **Restart counters:** pods show restarts from the low-CPU period in Lab 9. They are not related to this lab.

## Conclusion
A taint keeps pods off a node unless they tolerate it, and a label marks the node so pods can target it. Both were applied and verified on `worker-1`.