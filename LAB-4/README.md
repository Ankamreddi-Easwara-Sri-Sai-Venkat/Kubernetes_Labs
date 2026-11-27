
# Experiment 4: Read the Scheduler and Find Why Pods Stay Pending

**Course:** 26CS52B1E – Kubernetes and Container Orchestration

## Aim

Read the Kubernetes scheduler source code, identify the default filter and score plugins and their weights, and deliberately create Pending Pods to match each `FailedScheduling` message to the plugin that produced it.

## Software Requirements

- Docker Desktop (Engine running)
- kind
- kubectl
- Git
- Windows PowerShell
- Notepad or Visual Studio Code
- Three-node kind cluster named `klu`

## Project Structure

```text
Experiment4_Scheduler_Lab/
│
├── README.md
├── commands.txt
│
├── big.yaml
├── ssd.yaml
├── spread-tol.yaml
└── hostport.yaml
```

## 1. Check the Cluster

Start Docker Desktop and open PowerShell.

```powershell
kubectl config use-context kind-klu
kubectl get nodes
kubectl describe node klu-control-plane | Select-String "Taints"
git --version
```

If Git is not installed:

```powershell
winget install Git.Git
```

Close and reopen PowerShell after installation.

## 2. Download Kubernetes Scheduler Source Code

```powershell
cd $HOME

git clone --depth 1 --branch v1.37.0 --filter=blob:none --sparse https://github.com/kubernetes/kubernetes.git k8s-src

cd k8s-src

git sparse-checkout set pkg/scheduler staging/src/k8s.io/kube-scheduler

dir pkg\scheduler
```

This downloads only the scheduler source folders required for the experiment.

## 3. Inspect Scheduler Plugins

Open the default plugin configuration:

```powershell
notepad pkg\scheduler\apis\config\v1\default_plugins.go
```

List the default plugins:

```powershell
Select-String -Path pkg\scheduler\apis\config\v1\default_plugins.go -Pattern "Name: names"
```

Find Filter and Score functions:

```powershell
Get-ChildItem pkg\scheduler\framework\plugins -Recurse -Filter *.go -Exclude *_test.go | Select-String -Pattern "^func \(.*\) (Filter|Score)\("
```

Find scheduling-cycle functions:

```powershell
Select-String -Path pkg\scheduler\schedule_one.go -Pattern "^func .*(ScheduleOne|schedulingCycle|bindingCycle|schedulePod|findNodesThatFitPod|prioritizeNodes)\("
```

### Default Score Plugin Weights

| Plugin | Weight |
|---|---:|
| TaintToleration | 3 |
| NodeAffinity | 2 |
| PodTopologySpread | 2 |
| NodeResourcesFit | 1 |

Use your actual source-code output to complete the plugin table.

## 4. Create the Lab Folder

```powershell
mkdir $HOME\lab4
cd $HOME\lab4
```

Save all YAML files in this folder.

## 5. Experiment A – Insufficient CPU

Apply the Pod:

```powershell
kubectl apply -f big.yaml
kubectl get pod big
kubectl describe pod big
```

Find the error in the scheduler source:

```powershell
Select-String -Path $HOME\k8s-src\pkg\scheduler\framework\plugins\noderesources\fit.go -Pattern "Insufficient"
```

**Expected result:** Pod remains Pending because the requested CPU is unavailable.

**Scheduler Plugin:** NodeResourcesFit.

## 6. Experiment B – Missing Node Label

Apply the Pod:

```powershell
kubectl apply -f ssd.yaml
kubectl describe pod needs-ssd
```

Add the required label:

```powershell
kubectl label node klu-worker disktype=ssd
kubectl get pod needs-ssd -o wide
```

Find the message in source code:

```powershell
Select-String -Path $HOME\k8s-src\pkg\scheduler\framework\plugins\nodeaffinity\node_affinity.go -Pattern "didn't match"
```

**Expected result:** Pod schedules after adding the required node label.

**Scheduler Plugin:** NodeAffinity.

## 7. Experiment C – Taints and Tolerations

Add a taint:

```powershell
kubectl taint node klu-worker exam=running:NoSchedule
```

Create a Deployment:

```powershell
kubectl create deployment spread --image=nginx:1.27 --replicas=4

kubectl get pods -l app=spread -o wide
```

Apply the toleration configuration:

```powershell
kubectl apply -f spread-tol.yaml

kubectl get pods -l app=spread -o wide
```

Remove the taint:

```powershell
kubectl taint node klu-worker exam=running:NoSchedule-
```

**Expected result:** Pods can schedule on the tainted worker when they have the matching toleration.

**Scheduler Plugin:** TaintToleration.

## 8. Experiment D – Host Port Conflict

Apply the Deployment:

```powershell
kubectl apply -f hostport.yaml

kubectl get pods -l app=hostport -o wide
```

Describe the Pending Pod using its actual name:

```powershell
kubectl describe pod YOUR_PENDING_POD_NAME
```

Find the message in the source code:

```powershell
Select-String -Path $HOME\k8s-src\pkg\scheduler\framework\plugins\nodeports\node_ports.go -Pattern "free ports"
```

**Expected result:** Two Pods run on the two workers, and the third remains Pending because port 8080 is already occupied on both workers.

**Scheduler Plugin:** NodePorts.

## 9. Experiment E – Cordon and Uncordon Nodes

Cordon both workers:

```powershell
kubectl cordon klu-worker
kubectl cordon klu-worker2
```

Check node status:

```powershell
kubectl get nodes
```

Create a Pod:

```powershell
kubectl run closed --image=nginx:1.27
kubectl describe pod closed
```

Uncordon both workers:

```powershell
kubectl uncordon klu-worker
kubectl uncordon klu-worker2
```

Check the Pod:

```powershell
kubectl get pod closed -o wide
```

**Expected result:** Pod initially remains Pending and schedules after the workers are uncordoned.

**Scheduler Plugin:** NodeUnschedulable.

## 10. Expected Plugin-to-Event Mapping

| Experiment | Scheduling Message | Plugin |
|---|---|---|
| A | Insufficient cpu | NodeResourcesFit |
| B | Didn't match Pod's node affinity/selector | NodeAffinity |
| C | Untolerated taint | TaintToleration |
| D | Didn't have free ports | NodePorts |
| E | Node(s) were unschedulable | NodeUnschedulable |

## 11. Cleanup

Delete Pods:

```powershell
kubectl delete pod big needs-ssd closed --ignore-not-found
```

Delete Deployments:

```powershell
kubectl delete deployment spread hostport --ignore-not-found
```

Remove the label:

```powershell
kubectl label node klu-worker disktype-
```

Remove the taint:

```powershell
kubectl taint node klu-worker exam=running:NoSchedule-
```

Uncordon workers:

```powershell
kubectl uncordon klu-worker
kubectl uncordon klu-worker2
```

Check remaining Pods:

```powershell
kubectl get pods
```

Keep the `k8s-src` folder for future experiments.

## 12. Evidence to Capture

1. Kubernetes nodes and control-plane taint.
2. Scheduler source folder.
3. Default scheduler plugin list.
4. Filter and Score functions.
5. Scheduling-cycle functions.
6. Experiment A Pending Pod.
7. Experiment B before and after adding the label.
8. Experiment C before and after toleration.
9. Experiment D Pending Pod.
10. Experiment E before and after uncordoning.

## Conclusion

This experiment demonstrates how Kubernetes scheduling works using Filter and Score plugins. Different scheduling constraints such as CPU requests, node labels, taints, host ports, and unschedulable nodes cause Pods to remain Pending. By examining the scheduler events and source code, we can identify which plugin rejects a node and understand how Kubernetes handles scheduling decisions.
