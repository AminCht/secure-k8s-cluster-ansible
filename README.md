# Secure Kubernetes Cluster Deployment via Ansible

This repository contains a **hardened, production-ready Ansible playbook** to deploy a Kubernetes cluster on **bare‑metal servers** or **VPS instances** (e.g., Hetzner, DigitalOcean) that communicate over **public networks**.

Unlike standard `kubeadm` setups, this project focuses heavily on **security**, ensuring that nodes can communicate safely over the internet **without a private network (VPC/LAN)** by using strict **firewall rules** and **encryption**.

---

## 🚀 Key Features & Security

### 🛡️ Strict Firewalling (UFW Whitelisting)

* All ports are **closed by default**
* Kubernetes ports (`6443`, `10250`, `2379`, etc.) are opened **only** for the specific IP addresses of your cluster nodes
* External access to the Kubernetes API or Kubelet is blocked

### 🔒 Encrypted Pod Network (WireGuard)

* Uses **Calico CNI** with **WireGuard** enabled
* All pod‑to‑pod traffic across nodes is **fully encrypted**
* Safe to run Kubernetes over public networks

### 🤖 Automated Kubelet Hardening

* Disables **anonymous authentication** on Kubelet
* Prevents unauthorized access to node APIs

### ⚡ Simple Scaling

* Add new worker nodes by simply updating the inventory file
* Ansible idempotency ensures existing nodes are not modified unnecessarily

---

## 📋 Prerequisites

* **Control Node**: Machine with Ansible installed
* **Target Nodes**: Ubuntu `20.04` or `22.04`
* **SSH Access**: Root or sudo user with **passwordless SSH (key‑based login)**

---

## 🛠️ Quick Start

### 1️⃣ Clone the Repository

```bash
git clone https://github.com/AminCht/secure-k8s-cluster-ansible.git
cd secure-k8s-cluster-ansible
```

---

### 2️⃣ Configure Inventory

Copy the example inventory file:

```bash
cp inventory.example.yml inventory.yml
```

Edit `inventory.yml` and replace the placeholder IPs with your actual server IPs:

```yaml
all:
  vars:
    ansible_user: root

  children:
    # --- Define Hosts ---
    Server_Master:
      hosts:
        1.1.1.1:   # <--- Master IP

    Server_Workers:
      hosts:
        2.2.2.2:   # <--- Worker 1 IP
        3.3.3.3:   # <--- Worker 2 IP (Optional)

    # --- Grouping (Do not change) ---
    masters:
      children:
        Server_Master:

    workers:
      children:
        Server_Workers:

    k8s_cluster:
      children:
        masters:
        workers:
```

---

### 3️⃣ Run the Playbook

Deploy the entire cluster with a single command:

```bash
ansible-playbook playbook/kubernetes.yml
```

The playbook will:

* Configure firewalls
* Install container runtime and Kubernetes components
* Initialize the control plane
* Join worker nodes securely

---

## 📈 Adding More Worker Nodes

Scaling the cluster is extremely simple:

1. Open `inventory.yml`
2. Add the new node IP under `Server_Workers`

```yaml
Server_Workers:
  hosts:
    2.2.2.2:
    3.3.3.3:
    4.4.4.4:   # <--- New Worker Node
```

3. Run the playbook again:

```bash
ansible-playbook -i inventory.yml kubernetes.yml
```

✅ Ansible is **idempotent** — only the new node will be configured, and firewall rules will be updated automatically.

---

## 🔍 Deep Dive: Architecture

### 🌐 The "Public Network" Challenge

Running Kubernetes on providers like Hetzner (without a vSwitch) or across multiple cloud providers means nodes communicate via **public IPs**, which introduces two major risks:

* **Unauthorized Access** – Attackers scanning common Kubernetes ports (`10250`, `6443`, etc.)
* **Man‑in‑the‑Middle Attacks** – Unencrypted pod traffic over the internet

---

### ✅ How This Project Solves It

#### 🔥 Dynamic UFW Rules

* Firewall rules are generated dynamically from the Ansible inventory
* Master node allows API access **only from worker IPs**
* Worker nodes allow Kubelet access **only from the master IP**

#### 🔐 Calico + WireGuard

* Calico is installed as the CNI
* WireGuard is enabled to encrypt **all pod‑to‑pod traffic**
* The public internet effectively becomes a secure private network

---

## ⚠️ Important Notes

* **SSH Port**: If you use a non‑standard SSH port, update the `security.yml` tasks to avoid locking yourself out
* **Resetting the Cluster**:

  * You can run `kubeadm reset` on nodes
  * For production environments, a full OS rebuild is strongly recommended

---

## 🤝 Contributing

Pull requests are welcome!

For major changes, please open an issue first to discuss your ideas.

## License
MIT
