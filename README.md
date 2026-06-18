# ☸️ kubectl Zero-To-Hero — A Hands-On Learning in 10 Days Roadmap

> **Learn kubectl from zero to production-ready in 10 focused days.**  
> Each day has a clear goal, key concepts, commands to practice, and a checkpoint task to commit to your repo.

-----

## 📋 Table of Contents

- [Prerequisites](#prerequisites)
- [How to Use This Guide](#how-to-use-this-guide)
- [Roadmap Overview](#roadmap-overview)
- [Day 1 – Why kubectl Exists](#day-1--why-kubectl-exists)
- [Day 2 – First kubectl Commands](#day-2--first-kubectl-commands)
- [Day 3 – Pods (The Core Unit)](#day-3--pods-the-core-unit)
- [Day 4 – Deployments & ReplicaSets](#day-4--deployments--replicasets)
- [Day 5 – Services & Networking](#day-5--services--networking)
- [Day 6 – Config & Secrets](#day-6--config--secrets)
- [Day 7 – Debugging Like a Pro](#day-7--debugging-like-a-pro)
- [Day 8 – YAML Mastery](#day-8--yaml-mastery)
- [Day 9 – Multiple Environments](#day-9--working-with-multiple-environments)
- [Day 10 – Mini Project](#day-10--mini-project)
- [Progress Tracker](#progress-tracker)
- [Useful Resources](#useful-resources)

-----

## Prerequisites

You don’t need Kubernetes experience. You just need:

- A terminal (Linux/macOS/WSL on Windows)
- Basic command-line comfort (`cd`, `ls`, `cat`)
- A cluster to practice on — pick one:
  - [Minikube](https://minikube.sigs.k8s.io/docs/start/) — runs locally on your machine
  - [Kind](https://kind.sigs.k8s.io/) — Kubernetes in Docker
  - [k3s](https://k3s.io/) — lightweight, great for VMs
  - [Play with Kubernetes](https://labs.play-with-k8s.com/) — free browser-based cluster

-----

## How to Use This Guide

1. **Fork or clone this repo** to track your progress
1. **Work through one day at a time** — don’t skip ahead
1. **Run every command yourself** — reading is not enough
1. **Complete the checkpoint** at the end of each day
1. **Commit your work** — push your YAML files and notes to this repo as you go

### Suggested repo structure as you progress:

```
kubectl-learning/
├── README.md
├── day-01/
│   └── notes.md
├── day-02/
│   └── notes.md
├── day-03/
│   ├── my-first-pod.yaml
│   └── notes.md
├── day-04/
│   ├── deployment.yaml
│   └── notes.md
...
└── day-10/
    ├── deployment.yaml
    ├── service.yaml
    ├── configmap.yaml
    ├── secret.yaml
    └── notes.md
```

-----

## Roadmap Overview

|Day|Topic                    |Goal                              |
|---|-------------------------|----------------------------------|
|1  |Why kubectl exists       |Understand the problem it solves  |
|2  |First commands           |Get comfortable with the CLI      |
|3  |Pods                     |Understand what actually runs     |
|4  |Deployments & ReplicaSets|Run apps the Kubernetes way       |
|5  |Services & Networking    |Make apps reachable               |
|6  |Config & Secrets         |Stop hardcoding configs           |
|7  |Debugging                |Be useful during outages          |
|8  |YAML mastery             |Read & write manifests confidently|
|9  |Multiple environments    |Real-world kubectl usage          |
|10 |Mini project             |Cement everything                 |

-----

## Day 1 – Why kubectl Exists

**Goal:** Understand the problem kubectl solves before touching any commands.

### Concepts

- **Kubernetes** is a system that manages containers across multiple machines. It has two parts:
  - **Control plane** — the brain (API server, scheduler, etcd)
  - **Nodes** — the workers that actually run your containers
- **kubectl** is a command-line tool that talks to the Kubernetes API server. It’s how you tell Kubernetes what you want.
- **Declarative vs Imperative:**
  - *Imperative* — “Do this now” (`kubectl run`, `kubectl delete`)
  - *Declarative* — “Here’s what I want, make it so” (`kubectl apply -f file.yaml`)
  - Kubernetes prefers declarative. So should you.
- **kubectl vs other tools:**
  - Kubernetes Dashboard — GUI, good for viewing, not for automation
  - Helm — package manager built on top of kubectl
  - GitOps tools (ArgoCD, Flux) — apply YAML from Git automatically; kubectl is still under the hood

### Setup

**Install kubectl:**

```bash
# macOS
brew install kubectl

# Linux
curl -LO "https://dl.k8s.io/release/$(curl -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl && sudo mv kubectl /usr/local/bin/

# Verify
kubectl version --client
```

**Start a local cluster (choose one):**

```bash
# Minikube
minikube start

# Kind
kind create cluster

# Verify cluster access
kubectl cluster-info
kubectl get nodes
```

### ✅ Day 1 Checkpoint

- [ ] kubectl installed and `kubectl version --client` shows output
- [ ] Cluster running and `kubectl get nodes` shows a Ready node
- [ ] Create `day-01/notes.md` explaining in your own words: what is the API server, and why does kubectl talk to it?

-----

## Day 2 – First kubectl Commands

**Goal:** Get comfortable navigating a cluster from the CLI.

### Core Commands

```bash
# List resources
kubectl get pods
kubectl get nodes
kubectl get all

# Get more details
kubectl get pods -o wide          # shows IP, node
kubectl get pods -o yaml          # full YAML output

# Inspect a specific resource
kubectl describe pod <pod-name>   # human-readable details + events

# Understand any resource
kubectl explain pod
kubectl explain pod.spec
kubectl explain deployment.spec.template
```

### Namespaces

Namespaces are like folders — they group and isolate resources.

```bash
kubectl get namespaces
kubectl get pods --namespace kube-system   # system pods
kubectl get pods -n kube-system            # shorthand
kubectl get pods --all-namespaces          # everything
```

### Contexts & Clusters

A **context** = cluster + user + namespace. Useful when you have multiple clusters.

```bash
kubectl config view                        # see all contexts
kubectl config current-context            # which context is active
kubectl config use-context <name>         # switch context
kubectl config get-contexts               # list all contexts
```

### ✅ Day 2 Checkpoint

- [ ] Run `kubectl get all -n kube-system` and understand what you see
- [ ] Use `kubectl explain` on at least 3 different resources
- [ ] Create `day-02/notes.md`: What’s the difference between `get` and `describe`?

-----

## Day 3 – Pods (The Core Unit)

**Goal:** Understand what actually runs in Kubernetes.

### What is a Pod?

- The **smallest deployable unit** in Kubernetes
- Wraps one or more containers that share network and storage
- Pods are **ephemeral** — they die, they don’t come back on their own
- You rarely create pods directly in production (use Deployments instead)

### Create a Pod — Imperative

```bash
# Run a pod quickly
kubectl run my-pod --image=nginx

# Check it
kubectl get pods
kubectl describe pod my-pod
```

### Create a Pod — Declarative (YAML)

Save this as `day-03/my-first-pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
    app: demo
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

```bash
# Apply it
kubectl apply -f day-03/my-first-pod.yaml

# Difference between apply and create:
# apply  — creates if not exists, updates if exists (idempotent)
# create — fails if resource already exists
```

### Pod Lifecycle

|Phase    |Meaning                                 |
|---------|----------------------------------------|
|Pending  |Scheduled but container not started yet |
|Running  |At least one container is running       |
|Succeeded|All containers exited successfully      |
|Failed   |At least one container exited with error|
|Unknown  |Node communication lost                 |

### Logs & Exec

```bash
# View logs
kubectl logs my-pod
kubectl logs my-pod -f                    # follow (stream live)
kubectl logs my-pod --previous            # logs from last crashed container

# Execute commands inside a container
kubectl exec -it my-pod -- /bin/bash
kubectl exec -it my-pod -- ls /etc/nginx
kubectl exec my-pod -- cat /etc/hosts

# Clean up
kubectl delete pod my-pod
kubectl delete -f day-03/my-first-pod.yaml
```

### ✅ Day 3 Checkpoint

- [ ] Create a pod using YAML and commit the file to `day-03/`
- [ ] Shell into the pod and run a command
- [ ] Create `day-03/notes.md`: Why are pods ephemeral, and why does that matter?

-----

## Day 4 – Deployments & ReplicaSets

**Goal:** Run apps the right way in Kubernetes.

### Why Not Just Use Pods?

|Problem                |What Happens                        |
|-----------------------|------------------------------------|
|Pod crashes            |It’s gone — nothing restarts it     |
|Need 3 copies          |You have to create 3 pods manually  |
|Want to update your app|You have to delete and recreate pods|

**Deployments** solve all of this.

### How It Works

```
Deployment
  └── manages ReplicaSet
        └── manages Pods (n replicas)
```

A **ReplicaSet** ensures N copies of a pod are always running. A **Deployment** manages ReplicaSets and handles updates.

### Create a Deployment

```bash
# Imperative
kubectl create deployment my-app --image=nginx --replicas=3

# Check what was created
kubectl get deployments
kubectl get replicasets
kubectl get pods
```

Save as `day-04/deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
```

```bash
kubectl apply -f day-04/deployment.yaml
```

### Scaling

```bash
kubectl scale deployment my-app --replicas=5
kubectl get pods    # watch 5 pods come up
```

### Rolling Updates & Rollbacks

```bash
# Update the image (triggers a rolling update)
kubectl set image deployment/my-app nginx=nginx:1.26

# Watch the rollout
kubectl rollout status deployment/my-app

# See history
kubectl rollout history deployment/my-app

# Undo last update
kubectl rollout undo deployment/my-app

# Undo to a specific revision
kubectl rollout undo deployment/my-app --to-revision=1
```

### ✅ Day 4 Checkpoint

- [ ] Deploy 3 replicas, then scale to 5
- [ ] Do a rolling update and watch `kubectl rollout status`
- [ ] Roll back and confirm the old image is running
- [ ] Commit `day-04/deployment.yaml`

-----

## Day 5 – Services & Networking

**Goal:** Make your application reachable.

### The Problem

Pods have IP addresses, but they change when pods restart. You need a **stable endpoint** to reach your app. That’s what a **Service** is.

### Service Types

|Type          |Access                              |When to Use           |
|--------------|------------------------------------|----------------------|
|`ClusterIP`   |Inside cluster only (default)       |Internal microservices|
|`NodePort`    |Via node’s IP + a port (30000-32767)|Local dev / testing   |
|`LoadBalancer`|External IP via cloud load balancer |Production on cloud   |

### How Services Find Pods

Services use **label selectors** to find pods. If a pod has the label `app: my-app`, a service with selector `app: my-app` will route traffic to it.

### Expose a Deployment

```bash
# Imperative
kubectl expose deployment my-app --port=80 --type=NodePort

# Check it
kubectl get svc
kubectl describe svc my-app
```

Save as `day-05/service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-svc
spec:
  selector:
    app: my-app        # matches pods with this label
  ports:
  - port: 80           # service port
    targetPort: 80     # container port
  type: NodePort
```

```bash
kubectl apply -f day-05/service.yaml

# On Minikube, get the URL
minikube service my-app-svc --url

# Port-forward (works on any cluster)
kubectl port-forward svc/my-app-svc 8080:80
# Now open http://localhost:8080
```

### ✅ Day 5 Checkpoint

- [ ] Expose your deployment and reach it from your browser or `curl`
- [ ] Commit `day-05/service.yaml`
- [ ] Create `day-05/notes.md`: What would happen if a pod’s label didn’t match the service selector?

-----

## Day 6 – Config & Secrets

**Goal:** Never hardcode configuration again.

### ConfigMaps

Store non-sensitive config (URLs, feature flags, settings).

```bash
# Create from literal values
kubectl create configmap app-config \
  --from-literal=APP_ENV=production \
  --from-literal=LOG_LEVEL=info

# Create from a file
kubectl create configmap app-config --from-file=config.properties

# View it
kubectl get configmap app-config -o yaml
```

Save as `day-06/configmap.yaml`:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: "production"
  LOG_LEVEL: "info"
  DATABASE_URL: "postgres://db-service:5432/mydb"
```

### Secrets

Store sensitive data (passwords, tokens, API keys). Base64-encoded, not encrypted by default — but can be encrypted at rest.

```bash
# Create a secret
kubectl create secret generic db-secret \
  --from-literal=DB_PASSWORD=supersecret \
  --from-literal=API_KEY=myapikey123

# View it (values are base64 encoded)
kubectl get secret db-secret -o yaml

# Decode a value
kubectl get secret db-secret -o jsonpath='{.data.DB_PASSWORD}' | base64 -d
```

### Use Config in a Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: app
        image: nginx
        env:
        # From ConfigMap
        - name: APP_ENV
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: APP_ENV
        # From Secret
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: DB_PASSWORD
```

### ✅ Day 6 Checkpoint

- [ ] Create a ConfigMap and a Secret
- [ ] Deploy an app that reads from both
- [ ] Exec into the pod and verify: `kubectl exec -it <pod> -- env | grep APP_ENV`
- [ ] Commit `day-06/configmap.yaml` (never commit real secrets to Git!)

-----

## Day 7 – Debugging Like a Pro

**Goal:** Be useful when things break.

### The Debug Workflow

```
1. kubectl get pods           → Is the pod running?
2. kubectl describe pod       → What events happened?
3. kubectl logs               → What did the app say?
4. kubectl exec               → Can I get inside?
```

### Common Problems & How to Fix Them

**CrashLoopBackOff**

```bash
kubectl describe pod <pod-name>   # look at Events section
kubectl logs <pod-name>           # app crash logs
kubectl logs <pod-name> --previous # logs from before last crash
```

*Causes:* app error on startup, bad config, missing env vars, wrong command

**ImagePullBackOff / ErrImagePull**

```bash
kubectl describe pod <pod-name>   # check Events for image pull error
```

*Causes:* wrong image name, wrong tag, private registry without credentials

**Pending Pod (never starts)**

```bash
kubectl describe pod <pod-name>   # look for "Insufficient CPU/memory"
kubectl get nodes                 # are nodes Ready?
```

*Causes:* not enough resources, node selector mismatch, taints

**OOMKilled (Out of Memory)**

```bash
kubectl describe pod <pod-name>   # look for OOMKilled in Last State
```

*Causes:* memory limit too low, memory leak in app

### Useful Debug Commands

```bash
# Watch pods in real time
kubectl get pods -w

# Get events for the whole namespace
kubectl get events --sort-by=.lastTimestamp

# Describe a node (check capacity)
kubectl describe node <node-name>

# Run a temporary debug pod
kubectl run debug --image=busybox --rm -it --restart=Never -- sh

# Check resource usage (requires metrics-server)
kubectl top pods
kubectl top nodes
```

### ✅ Day 7 Checkpoint

- [ ] Deliberately break a deployment (wrong image tag, e.g. `nginx:doesnotexist`)
- [ ] Debug it using only `describe`, `logs`, and `events`
- [ ] Fix it with a correct image and do a rollout
- [ ] Create `day-07/notes.md`: Walk through your debug session step by step

-----

## Day 8 – YAML Mastery

**Goal:** Read and write Kubernetes manifests confidently.

### The 4 Top-Level Fields (Every Resource Has These)

```yaml
apiVersion: apps/v1   # API group + version
kind: Deployment      # what type of resource
metadata:             # name, namespace, labels, annotations
  name: my-app
  namespace: default
  labels:
    app: my-app
    env: production
  annotations:
    description: "My production app"
spec:                 # desired state — differs per resource type
  ...
```

### Labels vs Annotations

|         |Labels               |Annotations             |
|---------|---------------------|------------------------|
|Purpose  |Selection & grouping |Metadata & documentation|
|Used by  |Selectors, services  |Tools, humans           |
|Queryable|Yes (`-l app=my-app`)|No                      |

### Resource Requests & Limits

```yaml
spec:
  containers:
  - name: app
    image: nginx
    resources:
      requests:          # minimum guaranteed
        cpu: "100m"      # 100 millicores = 0.1 CPU
        memory: "128Mi"
      limits:            # maximum allowed
        cpu: "500m"
        memory: "256Mi"
```

### Liveness & Readiness Probes

```yaml
spec:
  containers:
  - name: app
    image: nginx
    livenessProbe:       # if this fails → restart container
      httpGet:
        path: /healthz
        port: 80
      initialDelaySeconds: 10
      periodSeconds: 5
    readinessProbe:      # if this fails → remove from Service endpoints
      httpGet:
        path: /ready
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 3
```

### Validate Your YAML

```bash
# Dry run — check without applying
kubectl apply -f my-file.yaml --dry-run=client

# Validate server-side
kubectl apply -f my-file.yaml --dry-run=server

# Diff what would change
kubectl diff -f my-file.yaml
```

### ✅ Day 8 Checkpoint

- [ ] Write a complete Deployment YAML from scratch (no copy-paste)
- [ ] Include: resource limits, a liveness probe, labels, annotations
- [ ] Validate with `--dry-run=client` before applying
- [ ] Commit as `day-08/deployment-with-probes.yaml`

-----

## Day 9 – Working with Multiple Environments

**Goal:** Manage apps across dev, staging, and production.

### Namespaces for Environment Separation

```bash
# Create namespaces
kubectl create namespace dev
kubectl create namespace staging
kubectl create namespace production

# Deploy to a specific namespace
kubectl apply -f deployment.yaml -n dev
kubectl apply -f deployment.yaml -n staging

# View resources per namespace
kubectl get all -n dev
kubectl get all -n staging
```

### Context Switching

A **kubeconfig** file (usually at `~/.kube/config`) holds all your cluster connections.

```bash
# View all contexts
kubectl config get-contexts

# Switch context
kubectl config use-context my-prod-cluster

# Set a default namespace for a context
kubectl config set-context --current --namespace=dev

# Rename a context
kubectl config rename-context old-name new-name
```

**Tip:** Install `kubectx` and `kubens` for faster switching:

```bash
brew install kubectx     # includes both kubectx and kubens
kubectx                  # list contexts
kubectx my-prod          # switch cluster
kubens dev               # switch namespace
```

### Safe Cluster Access Practices

```bash
# Always check which context you're in before destructive operations
kubectl config current-context

# Use --dry-run before applying in production
kubectl apply -f deployment.yaml --dry-run=server -n production

# Use aliases to reduce mistakes
alias kdev='kubectl -n dev'
alias kprod='kubectl -n production'
```

### ✅ Day 9 Checkpoint

- [ ] Create `dev` and `staging` namespaces
- [ ] Deploy your app to both with different ConfigMap values
- [ ] Practice switching between them with `kubectx`/`kubens` or `kubectl config use-context`
- [ ] Create `day-09/notes.md`: How would you prevent someone from accidentally running `kubectl delete` in production?

-----

## Day 10 – Mini Project

**Goal:** Tie everything together into one real-world workflow.

### What You’ll Build

A complete application stack managed entirely with kubectl:

```
nginx (web server)
  ├── Deployment (3 replicas)
  ├── Service (NodePort)
  ├── ConfigMap (app settings)
  └── Secret (fake API key)
```

### Step 1 — Create the namespace

```bash
kubectl create namespace mini-project
```

### Step 2 — Apply all manifests

Save each file in `day-10/`, then:

```bash
kubectl apply -f day-10/ -n mini-project
```

**day-10/configmap.yaml**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_NAME: "My kubectl App"
  LOG_LEVEL: "info"
```

**day-10/secret.yaml**

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  API_KEY: bXlzZWNyZXRrZXkxMjM=   # base64 of "mysecretkey123"
```

**day-10/deployment.yaml**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
        env:
        - name: APP_NAME
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: APP_NAME
        - name: API_KEY
          valueFrom:
            secretKeyRef:
              name: app-secret
              key: API_KEY
        resources:
          requests:
            cpu: "100m"
            memory: "64Mi"
          limits:
            cpu: "200m"
            memory: "128Mi"
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
```

**day-10/service.yaml**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-svc
spec:
  selector:
    app: my-app
  ports:
  - port: 80
    targetPort: 80
  type: NodePort
```

### Step 3 — Verify everything

```bash
kubectl get all -n mini-project
kubectl port-forward svc/my-app-svc 8080:80 -n mini-project
curl http://localhost:8080
```

### Step 4 — Rolling update

```bash
kubectl set image deployment/my-app nginx=nginx:1.26 -n mini-project
kubectl rollout status deployment/my-app -n mini-project
```

### Step 5 — Break it and debug it

```bash
# Set a bad image
kubectl set image deployment/my-app nginx=nginx:thisdoesnotexist -n mini-project

# Debug it
kubectl get pods -n mini-project
kubectl describe pod <pod-name> -n mini-project

# Roll back
kubectl rollout undo deployment/my-app -n mini-project
kubectl rollout status deployment/my-app -n mini-project
```

### ✅ Day 10 Checkpoint

- [ ] All 4 manifests applied and app is reachable
- [ ] Rolling update completed successfully
- [ ] Deliberately broken, debugged, and rolled back
- [ ] Everything committed to `day-10/`
- [ ] Write a short `day-10/notes.md` — what was the hardest part? What clicked?

-----

## Progress Tracker

Update this as you go:

|Day|Topic             |Status       |Notes|
|---|------------------|-------------|-----|
|1  |Why kubectl exists|⬜ Not started|     |
|2  |First commands    |⬜ Not started|     |
|3  |Pods              |⬜ Not started|     |
|4  |Deployments       |⬜ Not started|     |
|5  |Services          |⬜ Not started|     |
|6  |Config & Secrets  |⬜ Not started|     |
|7  |Debugging         |⬜ Not started|     |
|8  |YAML mastery      |⬜ Not started|     |
|9  |Multi-environment |⬜ Not started|     |
|10 |Mini project      |⬜ Not started|     |


> Replace ⬜ with ✅ as you complete each day.

-----

## What You’ll Know After This Roadmap

- ✅ Confident with kubectl in daily use
- ✅ Understand Kubernetes primitives (Pod, Deployment, Service, ConfigMap, Secret)
- ✅ Debug real production issues systematically
- ✅ Manage apps across multiple environments
- ✅ Write clean, production-ready YAML
- ✅ Ready for real Kubernetes work

-----

## Useful Resources

|Resource                       |Link                                                    |
|-------------------------------|--------------------------------------------------------|
|Official kubectl docs          |https://kubernetes.io/docs/reference/kubectl/           |
|kubectl cheat sheet            |https://kubernetes.io/docs/reference/kubectl/cheatsheet/|
|Kubernetes concepts            |https://kubernetes.io/docs/concepts/                    |
|Minikube getting started       |https://minikube.sigs.k8s.io/docs/start/                |
|Kind quickstart                |https://kind.sigs.k8s.io/docs/user/quick-start/         |
|Interactive Kubernetes tutorial|https://kubernetes.io/docs/tutorials/kubernetes-basics/ |
|kubectx/kubens                 |https://github.com/ahmetb/kubectx                       |

-----

## Quick Reference — Most Used Commands

```bash
# Get resources
kubectl get pods / nodes / deployments / svc / configmaps / secrets / all

# Inspect
kubectl describe <resource> <name>
kubectl logs <pod-name> [-f] [--previous]
kubectl exec -it <pod-name> -- /bin/bash

# Apply & manage
kubectl apply -f <file.yaml>
kubectl delete -f <file.yaml>
kubectl delete pod/svc/deployment <name>

# Deployments
kubectl scale deployment <name> --replicas=<n>
kubectl set image deployment/<name> <container>=<image>:<tag>
kubectl rollout status / history / undo deployment/<name>

# Namespaces & contexts
kubectl get all -n <namespace>
kubectl config use-context <name>
kubectl config set-context --current --namespace=<ns>

# Debug
kubectl get events --sort-by=.lastTimestamp
kubectl run debug --image=busybox --rm -it --restart=Never -- sh
kubectl apply -f file.yaml --dry-run=client
```

-----

*Good luck! The best way to learn kubectl is to break things, debug them, and fix them. This repo is your record of doing exactly that.*

-----

## ⭐ Support This Project

If this roadmap helped you, please consider **starring this repository** — it helps others find it and keeps the motivation going!

[![GitHub stars](https://img.shields.io/github/stars/swapnilmali101/kubectl-zero-to-hero?style=social)](https://github.com/swapnilmali101/kubectl-zero-to-hero)

-----

## 📄 License

This project is licensed under the **MIT License** — you are free to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of this material, provided the original author credit is retained. See the [LICENSE](./LICENSE) file for full details.

-----

## ✍️ Author

<!-- Start author box -->
<table align="center" width="100%">
    <tr>
        <td align="center" valign="middle" width="30%">
            <img src="https://github.com/swapnilmali101/swapnilmali101/blob/master/assets/swapnilmali-profile-pic.png" alt="Swapnil Mali" width="150" style="border-radius: 50%; max-width="100%;">
        </td>
        <td align="left" valign="top" width="70%">
            <h2>SWAPNIL MALI.</h2>
            <p>
                <a href="https://github.com/swapnilmali101" target="_blank" align="center">
                   <img src="https://img.shields.io/badge/GitHub-Profile-181717?style=for-the-badge&logo=github&logoColor=yellow" alt="GitHub Profile">
                </a>
            </p>
            <p><em>👨🏻‍💻CS Engineer | AWS & DevOps Specialist -🎯focused on building reliable, observable, and scalable systems.</em></p>
        </td>
    </tr>
</table>
<!-- End author box -->

---
