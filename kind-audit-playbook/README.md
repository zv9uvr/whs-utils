# kind Audit Playbook

- kind (Kubernetes IN Docker) is a tool for running local Kubernetes clusters using Docker container “nodes”.
- kind was primarily designed for testing Kubernetes itself, but may be used for local development or CI.
- kind uses `kubeadm` to create clusters, and it supports multi-node clusters.

<br/>

### Phase #1 : Setup Single Node Cluster & Deploy WordPress

- Install

```bash
# Mac OS
# via homebrew (https://brew.sh/)
brew install kind
```

```bash
# Windows
> Get-ExecutionPolicy
Restricted
> Set-ExecutionPolicy AllSigned
> Get-ExecutionPolicy
AllSigned

# Install Chocolatey (https://chocolatey.org/install)
> Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))

# If you don't see any errors, you are ready to use Chocolatey! Type choco or choco -?
```

- Make 1 single node cluster

```bash
kind create cluster --name kind-single-node
```

- kubectl cheetsheet

  - <https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands>
  - <https://kubernetes.io/ko/docs/reference/kubectl/cheatsheet/>

- Check config file

```bash
cat ~/.kube/config
kubectl config view
```

- Check cluster

```bash
kubectl cluster-info --context kind-kind-single-node
```

- Check nodes

```bash
kubectl get nodes
```

- Check pods

```bash
kubectl get pods --all-namespaces
```

- Check ALL

```bash
kubectl get all --all-namespaces
```

- Deploy WordPress App Using Helm

```bash
# Add Helm Repo
helm repo add bitnami https://charts.bitnami.com/bitnami

# Update Helm Repo
helm repo update

# Deploy WordPress
helm install my-wordpress bitnami/wordpress

# List your Helm Chart
helm list
```

- Check WordPress

```bash
kubectl get all
```

- Port Forwarding

```bash
kubectl port-forward svc/my-wordpress 8080:80
```

<br/>

## Phase #2 : K8s Security Scanning (CIS Benchmark)

### kube-bench Installation & Execution

`kube-bench` is a tool that checks whether Kubernetes is deployed securely by running the checks documented in the **CIS Kubernetes Benchmark**.  
Official GitHub: [aquasecurity/kube-bench](https://github.com/aquasecurity/kube-bench)

- Run as a Kubernetes Job:

```bash
# Download kube-bench job.yaml
curl -L https://raw.githubusercontent.com/aquasecurity/kube-bench/main/job.yaml -o job.yaml

# Apply job
kubectl apply -f job.yaml

#
kubectl get all

#
kubectl get pods

# Wait for a few seconds for the job to complete
$ kubectl get pods
NAME                            READY   STATUS      RESTARTS      AGE
kube-bench-kw2xk                0/1     Completed   0             43s

# The results are held in the pod's logs
kubectl logs kube-bench-kw2xk
...
== Summary policies ==
0 checks PASS
6 checks FAIL
29 checks WARN
0 checks INFO

== Summary total ==
61 checks PASS
17 checks FAIL
52 checks WARN
0 checks INFO
```

For more information and different ways to run kube-bench see [documentation.](https://github.com/aquasecurity/kube-bench/blob/main/docs/running.md)

<br/>

## Phase #3 : Docker Image Security Scanning (Trivy)

`Trivy` is an **all-in-one cloud native security scanner** developed by Aqua Security.  
It detects vulnerabilities in container images, file systems, and Git repositories, and also supports compliance scanning.

Official GitHub: [aquasecurity/trivy](https://github.com/aquasecurity/trivy)

### 1. Install Trivy (CLI)

```bash
# MacOS
brew install aquasecurity/trivy/trivy

# Linux (Debian/Ubuntu)
sudo apt-get install wget apt-transport-https gnupg lsb-release -y
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy -y
```

Verify installation:

```bash
trivy --version
```

<br/>

### 2. Scan a Docker Image

```bash
trivy image <image_name>
```

Example output:

```bash
trivy image kindest/node:v1.33.1
2025-08-18T13:20:08+09:00 INFO [vuln] Vulnerability scanning is enabled
2025-08-18T13:20:08+09:00 INFO [secret] Secret scanning is enabled
2025-08-18T13:20:08+09:00 INFO [secret] If your scanning is slow, please try '--scanners vuln' to disable secret scanning
2025-08-18T13:20:08+09:00 INFO [secret] Please see also https://trivy.dev/v0.65/docs/scanner/secret#recommendation for faster secret detection
2025-08-18T13:21:09+09:00 INFO Detected OS family="debian" version="12.11"
2025-08-18T13:21:09+09:00 INFO [debian] Detecting vulnerabilities... os_version="12" pkg_num=174
2025-08-18T13:21:09+09:00 INFO Number of language-specific files num=13
2025-08-18T13:21:09+09:00 INFO [gobinary] Detecting vulnerabilities...
2025-08-18T13:21:09+09:00 WARN Using severities from other vendors for some vulnerabilities. Read https://trivy.dev/v0.65/docs/scanner/vulnerability#severity-selection for details.

Report Summary

┌──────────────────────────────────────────────┬──────────┬─────────────────┬─────────┐
│                    Target                    │   Type   │ Vulnerabilities │ Secrets │
├──────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ kindest/node:v1.33.1 (debian 12.11)          │  debian  │       173       │    -    │
├──────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ opt/cni/bin/host-local                       │ gobinary │        4        │    -    │
├──────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ opt/cni/bin/loopback                         │ gobinary │        4        │    -    │
├──────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ opt/cni/bin/portmap                          │ gobinary │        4        │    -    │
├──────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ opt/cni/bin/ptp                              │ gobinary │        4        │    -    │
├──────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/bin/kubeadm                              │ gobinary │        5        │    -    │
├──────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/bin/kubectl                              │ gobinary │        5        │    -    │
├──────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/bin/kubelet                              │ gobinary │        6        │    -    │
├──────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/bin/containerd                     │ gobinary │        4        │    -    │
├──────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/bin/containerd-fuse-overlayfs-grpc │ gobinary │        7        │    -    │
├──────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/bin/containerd-shim-runc-v2        │ gobinary │        4        │    -    │
├──────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/bin/crictl                         │ gobinary │        4        │    -    │
├──────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/bin/ctr                            │ gobinary │        4        │    -    │
├──────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/sbin/runc                          │ gobinary │        6        │    -    │
└──────────────────────────────────────────────┴──────────┴─────────────────┴─────────┘
Legend:
- '-': Not scanned
- '0': Clean (no security findings detected)


kindest/node:v1.33.1 (debian 12.11)

Total: 173 (UNKNOWN: 1, LOW: 115, MEDIUM: 37, HIGH: 17, CRITICAL: 3)
...
```
