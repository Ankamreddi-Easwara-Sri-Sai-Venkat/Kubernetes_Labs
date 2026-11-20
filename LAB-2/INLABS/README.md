# LAB-2 — Experiment 2: Trace `kubectl create deployment` End to End

## Aim
To follow one Deployment from the `kubectl` command to running containers, and collect evidence of the component that acts at each step: API server, etcd, controllers, scheduler, kubelet and kube-proxy.

## Software required
- Docker Desktop (Engine running)
- kind
- kubectl
- Windows PowerShell
- The 3-node kind cluster `klu` created in LAB-1

## Folder contents
- `commands.txt` — commands to perform the experiment in order.
- `web-old.yaml` — temporary file generated during Step 8. It must be generated from the live cluster; do not use a hand-written or old copy.
- `Screenshots/` — save screenshots of the evidence listed below.

## Procedure
Follow `commands.txt` from Step 0 through Step 9. Use three PowerShell windows:
1. Events watch: `kubectl get events -w`
2. Pods watch: `kubectl get pods -o wide -w`
3. Main terminal: run the experiment commands.

## Screenshots to collect
1. Cluster nodes and the three terminal windows
2. API server POST request showing `201 Created`
3. Events with their source components
4. ReplicaSet/Pod ownership, node and Pod IP
5. etcd keys containing `web`
6. `crictl pods` and `crictl ps` output
7. Pod sandbox IP and kubelet log lines
8. Service, EndpointSlice and kube-proxy rules
9. The `409 Conflict` error

## Analysis and inference
Write the full Deployment story in order, one line per step, naming the component and the evidence collected. Then explain:
- Which components do not produce events
- How the scheduler and kubelet communicate without calling each other
- Why `kubectl replace` failed while `kubectl scale` succeeded

## Result
Write the result in 2–3 lines after completing the experiment.

## Important note about `web-old.yaml`
The file is deliberately generated during Step 8 using:

```powershell
cmd /c "kubectl get deployment web -o yaml > web-old.yaml"
```

This captures the current Deployment and its current `resourceVersion`. Immediately afterward, scale the Deployment and run `kubectl replace -f web-old.yaml` to demonstrate the stale-version `409 Conflict`. The generated file is temporary evidence for this step.
