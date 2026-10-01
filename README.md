# Kubernetes Cluster with Vagrant & kubeadm

A hands-on Kubernetes cluster deployment project using **Vagrant**, **VirtualBox**, **Linux**, and **kubeadm**.

This project automates the initial provisioning of a Kubernetes lab environment using a `Vagrantfile` and provisioning scripts, then initializes the Kubernetes Control Plane and joins worker nodes using `kubeadm`.

---

## Architecture

The cluster consists of one Control Plane and two Worker Nodes:

```text
                         Kubernetes Cluster
                                │
                                │
                    ┌──────────────────────┐
                    │    Control Plane     │
                    │      cont01          │
                    │  192.168.56.30       │
                    └──────────┬───────────┘
                               │
                 ┌─────────────┴─────────────┐
                 │                           │
        ┌──────────────────┐       ┌──────────────────┐
        │     Worker 01    │       │     Worker 02    │
        │      work01      │       │      work02      │
        │  192.168.56.31   │       │  192.168.56.32   │
        └──────────────────┘       └──────────────────┘
```

### Node Configuration

| Node     | Role          | IP Address      |
| -------- | ------------- | --------------- |
| `cont01` | Control Plane | `192.168.56.30` |
| `work01` | Worker        | `192.168.56.31` |
| `work02` | Worker        | `192.168.56.32` |

---

## Technologies

* Vagrant
* VirtualBox
* Ubuntu Linux
* Kubernetes
* kubeadm
* kubelet
* kubectl
* Containerd
* Bash
* Flannel CNI

---

# Project Structure

```text
.
├── Vagrantfile
├── scripts/
│   ├── common.sh
│   ├── controller.sh
│   └── worker.sh
└── README.md
```

The exact script names can be adjusted to match your repository.

---

# Prerequisites

Before starting, make sure the following are installed on the host machine:

* VirtualBox
* Vagrant
* Git

Verify the installations:

```bash
vagrant --version
```

```bash
VBoxManage --version
```

---

# 1. Initialize the Vagrant Project

Create the project directory:

```bash
mkdir kubernetes-kubeadm-lab
cd kubernetes-kubeadm-lab
```

Initialize Vagrant:

```bash
vagrant init
```

This creates the initial:

```text
Vagrantfile
```

The `Vagrantfile` defines the Kubernetes virtual machines, networking, resources, and provisioning scripts.

---

# 2. Provision the Kubernetes Nodes

The environment is provisioned using the `Vagrantfile` and shell scripts.

Start the complete environment:

```bash
vagrant up
```

Vagrant will:

1. Create the virtual machines.
2. Configure the private network.
3. Install the required Kubernetes dependencies.
4. Configure containerd.
5. Install kubeadm.
6. Install kubelet.
7. Install kubectl where required.
8. Prepare the nodes for Kubernetes.

After provisioning, verify the machines:

```bash
vagrant status
```

Expected architecture:

```text
cont01  -> 192.168.56.30
work01  -> 192.168.56.31
work02  -> 192.168.56.32
```

---

# 3. Control Plane Initialization

The Control Plane is already prepared by the provisioning process.

No additional manual configuration is required on the Control Plane before obtaining the worker join command.

Connect to the Control Plane:

```bash
vagrant ssh cont01
```

Verify the node:

```bash
hostname
```

Expected:

```text
cont01
```

Check Kubernetes:

```bash
kubectl version
```

Check the cluster:

```bash
kubectl get nodes
```

At this stage, the Control Plane should be initialized and ready for worker nodes to join.

---

# 4. Generate the Worker Join Command

On the Control Plane, generate a new kubeadm join command:

```bash
kubeadm token create --print-join-command
```

Example:

```bash
kubeadm join 192.168.56.30:6443 \
  --token xkcaee.cpfaa0q8monc0mhm \
  --discovery-token-ca-cert-hash sha256:bd24e8d6cc8a71aed20c77de33faf9ab950ae56911244d8dbd5c223757d21621
```

> **Important:** The token and certificate hash are dynamic. Do not hard-code the example command into scripts or documentation. Generate a new command whenever required.

The command tells a worker node:

* Which Kubernetes API Server to connect to.
* Which authentication token to use.
* Which CA certificate hash to trust.

---

# 5. Join Worker Nodes to the Cluster

Connect to Worker 01:

```bash
vagrant ssh work01
```

Run the join command generated from the Control Plane:

```bash
kubeadm join 192.168.56.30:6443 \
  --token <TOKEN> \
  --discovery-token-ca-cert-hash sha256:<HASH>
```

Repeat the process for Worker 02:

```bash
vagrant ssh work02
```

Then run the same generated join command:

```bash
kubeadm join 192.168.56.30:6443 \
  --token <TOKEN> \
  --discovery-token-ca-cert-hash sha256:<HASH>
```

After a successful join, kubeadm will report that the node has joined the cluster.

---

# 6. Verify the Kubernetes Cluster

Return to the Control Plane:

```bash
vagrant ssh cont01
```

Check all nodes:

```bash
kubectl get nodes -o wide
```

Expected result:

```text
NAME     STATUS   ROLES           INTERNAL-IP
cont01   Ready    control-plane   192.168.56.30
work01   Ready    <none>          192.168.56.31
work02   Ready    <none>          192.168.56.32
```

The important status is:

```text
Ready
```

All three nodes should eventually appear as `Ready`.

---

# 7. Install a Pod Network

Kubernetes requires a Container Network Interface (CNI) plugin so that pods can communicate with each other across nodes.

This project uses **Flannel**.

Apply the Flannel configuration from the Control Plane:

```bash
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

Verify the Flannel pods:

```bash
kubectl get pods -n kube-flannel
```

Verify all nodes:

```bash
kubectl get nodes
```

Expected:

```text
NAME     STATUS   ROLES           AGE
cont01   Ready    control-plane   ...
work01   Ready    <none>          ...
work02   Ready    <none>          ...
```

---

# 8. Verify Cluster Components

Check all Kubernetes system pods:

```bash
kubectl get pods -A
```

Check system namespaces:

```bash
kubectl get namespaces
```

Check cluster information:

```bash
kubectl cluster-info
```

Check node details:

```bash
kubectl describe node cont01
```

```bash
kubectl describe node work01
```

```bash
kubectl describe node work02
```

---

# 9. Test the Cluster

Create a test deployment:

```bash
kubectl create deployment nginx --image=nginx
```

Check the deployment:

```bash
kubectl get deployments
```

Check the pod:

```bash
kubectl get pods -o wide
```

Expose the deployment:

```bash
kubectl expose deployment nginx --port=80 --type=NodePort
```

Check the service:

```bash
kubectl get svc
```

Example:

```text
NAME    TYPE       CLUSTER-IP      PORT(S)
nginx   NodePort   10.x.x.x        80:30xxx/TCP
```

The NodePort can then be accessed through one of the Kubernetes nodes.

---

# 10. Useful Vagrant Commands

### Start the cluster

```bash
vagrant up
```

### Check VM status

```bash
vagrant status
```

### Connect to Control Plane

```bash
vagrant ssh cont01
```

### Connect to Worker 01

```bash
vagrant ssh work01
```

### Connect to Worker 02

```bash
vagrant ssh work02
```

### Stop the environment

```bash
vagrant halt
```

### Restart the environment

```bash
vagrant reload
```

### Destroy the complete lab

```bash
vagrant destroy -f
```

After destroying the environment, recreate it with:

```bash
vagrant up
```

---

# 11. Useful Kubernetes Commands

### Show nodes

```bash
kubectl get nodes
```

### Show detailed node information

```bash
kubectl get nodes -o wide
```

### Show all pods

```bash
kubectl get pods -A
```

### Show services

```bash
kubectl get svc
```

### Show deployments

```bash
kubectl get deployments
```

### Show namespaces

```bash
kubectl get namespaces
```

### Show cluster information

```bash
kubectl cluster-info
```

### Generate a new worker join command

```bash
kubeadm token create --print-join-command
```

---

# 12. Troubleshooting

## Worker Cannot Join the Cluster

Check connectivity from the worker:

```bash
ping 192.168.56.30
```

Check whether the Kubernetes API Server is reachable:

```bash
curl -k https://192.168.56.30:6443
```

Check kubelet:

```bash
systemctl status kubelet
```

Check containerd:

```bash
systemctl status containerd
```

---

## Worker Already Joined / Need to Rejoin

If a worker needs to be removed from the cluster:

```bash
sudo kubeadm reset -f
```

Then clean the CNI configuration if required:

```bash
sudo rm -rf /etc/cni/net.d
```

Restart services:

```bash
sudo systemctl restart containerd
sudo systemctl restart kubelet
```

Generate a new join command from the Control Plane:

```bash
kubeadm token create --print-join-command
```

Then run it on the worker.

---

# 13. Troubleshooting Node Status

If a node is not `Ready`:

```bash
kubectl get nodes
```

Get more information:

```bash
kubectl describe node <node-name>
```

Check kubelet:

```bash
sudo systemctl status kubelet
```

Check kubelet logs:

```bash
sudo journalctl -u kubelet -xe
```

Check containerd:

```bash
sudo systemctl status containerd
```

Check system pods:

```bash
kubectl get pods -A
```

---

# 14. What This Project Demonstrates

This project demonstrates practical experience with:

* Infrastructure provisioning using **Vagrant**
* Virtual machine management using **VirtualBox**
* Kubernetes cluster bootstrapping with **kubeadm**
* Kubernetes Control Plane configuration
* Kubernetes Worker Node configuration
* Node registration and cluster management
* Container runtime configuration using **containerd**
* Kubernetes networking using **Flannel**
* `kubectl` cluster administration
* Linux system administration
* Kubernetes troubleshooting
* Infrastructure automation using Bash
* Reproducible Kubernetes lab environments

---

# 15. Deployment Flow

The overall deployment process is:

```text
                    Vagrant
                       │
                       ▼
              Create Virtual Machines
                       │
                       ▼
             Run Provisioning Scripts
                       │
                       ▼
              Configure Containerd
                       │
                       ▼
          Install kubeadm / kubelet / kubectl
                       │
                       ▼
              Initialize Control Plane
                       │
                       ▼
        kubeadm token create --print-join-command
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
        Worker 01            Worker 02
             │                   │
             └─────────┬─────────┘
                       ▼
                 kubeadm join
                       │
                       ▼
               Kubernetes Cluster
                       │
                       ▼
                Install Flannel
                       │
                       ▼
                  All Nodes Ready
```

---

# 16. Future Improvements

Possible future improvements for this project:

* Automate `kubeadm init`
* Automatically generate and distribute the worker join command
* Automate worker node joining
* Add Helm
* Deploy Kubernetes Dashboard
* Add Prometheus and Grafana monitoring
* Add Jenkins CI/CD
* Deploy the VProfile application
* Add Harbor private container registry
* Implement Kubernetes application deployment through CI/CD

---

# Project Result

The final environment provides a reproducible Kubernetes cluster running on local virtual machines:

```text
┌──────────────────────────────────────────────┐
│             Kubernetes Cluster               │
│                                              │
│  ┌────────────────────────────────────────┐  │
│  │ Control Plane                          │  │
│  │ cont01                                 │  │
│  │ 192.168.56.30                          │  │
│  │ kube-apiserver / scheduler / etcd     │  │
│  └────────────────┬───────────────────────┘  │
│                   │                          │
│          ┌────────┴────────┐                 │
│          │                 │                 │
│  ┌───────▼────────┐ ┌──────▼─────────┐       │
│  │ Worker 01      │ │ Worker 02      │       │
│  │ work01         │ │ work02         │       │
│  │ 192.168.56.31  │ │ 192.168.56.32  │       │
│  └────────────────┘ └────────────────┘       │
│                                              │
└──────────────────────────────────────────────┘
```

The cluster can then be used as the foundation for deploying containerized applications, monitoring, CI/CD pipelines, and other Kubernetes workloads.
The final step 
I tried vproapp by yaml and create private repo (harbor) by helm
