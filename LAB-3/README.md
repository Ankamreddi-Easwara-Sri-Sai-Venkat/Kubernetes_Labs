
# Experiment 3: Write Your First Controller with client-go

## Course Details
- Course: 26CS52B1E
- Subject: Kubernetes and Container Orchestration
- Week: 2
- Experiment: 3

## Aim
To build and run a small Kubernetes controller in Go using client-go.

The controller ensures that every namespace labelled
klu.edu/managed=true contains a ConfigMap named welcome.

## Technologies Used
- Go 1.26+
- Kubernetes
- client-go
- Docker Desktop
- kind
- kubectl
- Windows PowerShell

## Project Structure

welcome-controller/
- main.go
- go.mod
- go.sum
- README.md
- commands.txt
- .gitignore

## Setup

```powershell
kubectl config use-context kind-klu
kubectl get nodes

mkdir $HOME\welcome-controller
cd $HOME\welcome-controller

go mod init welcome-controller

Copy-Item $HOME\Downloads\main.go .

go get k8s.io/client-go@v0.37.1 k8s.io/api@v0.37.1 k8s.io/apimachinery@v0.37.1

go mod tidy
go build .
```

## Run Controller

```powershell
go run .
```

## Experiments Performed

1. Creating a managed namespace.
2. Automatically creating a ConfigMap.
3. Testing self-healing after ConfigMap deletion.
4. Testing controller restart and recovery.
5. Running two controller instances.
6. Removing namespace labels.

## Cleanup

```powershell
kubectl delete namespace team-a team-b team-c
```

## Result

Successfully built and executed a Kubernetes controller
using Go client-go. Verified automatic ConfigMap creation,
self-healing, level-triggered reconciliation and idempotency.
