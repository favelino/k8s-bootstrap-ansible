# Kubernetes Day-0 Bootstrap with Ansible

Automated deployment of a Kubernetes cluster on Ubuntu virtual machines using **kubeadm** and **Ansible**.

This project focuses on repeatability, infrastructure as code, and production-style initialization of a control plane and worker nodes.

## Purpose

The goal of this project is to provide a deterministic and repeatable way to bootstrap Kubernetes clusters from scratch.

It demonstrates infrastructure automation practices used by SRE and platform engineering teams, enabling fast environment creation, reduced manual intervention, and consistent deployments across labs or production-like scenarios.

---

## Architecture

- 1 Control Plane node
- 2 Worker nodes
- Ubuntu OS
- containerd as container runtime
- Calico as CNI

---

## What this automation does

- [x] Prepare OS prerequisites
- [x] Install base dependencies
- [x] Install and configure containerd
- [x] Install kubeadm, kubelet, kubectl
- [x] Initialize the control plane
- [x] Generate and distribute join command
- [x] Join worker nodes
- [x] Install CNI networking

---

## Requirements

Before running the playbooks, ensure you have:

- Ansible installed on the control machine with a vault.
- SSH access to all nodes
- Sudo/root privileges
- Python available on target hosts
- Network connectivity between nodes

---

## Inventory

Edit the inventory file to reflect your environment.

Example:

```
[k8s_controller]
controller01 ansible_host=10.100.75.16

[k8s_workers]
worker01 ansible_host=10.100.75.17
worker02 ansible_host=10.100.75.18
```

---

## Variables

Key variables typically include:

- Pod network CIDR
- API server advertise address
- Kubernetes user
- CRI socket

Recommended approach: keep environment-specific variables in a file and pass it with `-e @file`.

Example:

```yaml
# envs/lab.yml
pod_network_cidr: "10.10.0.0/16"
apiserver_advertise_address: "10.100.75.16"
kube_admin_user: "fernando"
kube_admin_group: "fernando"
kube_admin_home: "/home/fernando"
```

Variables can also be defined in:

```
group_vars/
host_vars/
```

or overridden at runtime using `-e`.

---

## Vault

Sensitive information should be encrypted using Ansible Vault.

Example:

```bash
ansible-vault edit vault.yml
```

Run playbooks with:

```bash
--ask-vault-pass
```

or via a vault password file.

---

## Deployment Options

You can run the automation in two ways.

### Option A: Run in steps (recommended for first run and troubleshooting)

1. Base packages
```bash
ansible-playbook -i hosts.ini pre-install-base-packages.yml -e @envs/lab.yml --ask-vault-pass
```

2. OS/Kubernetes prerequisites
```bash
ansible-playbook -i hosts.ini k8s-part1-prep.yml -e @envs/lab.yml --ask-vault-pass
```

3. Container runtime (containerd)
```bash
ansible-playbook -i hosts.ini k8s-part2-containerd_v2.yml -e @envs/lab.yml --ask-vault-pass
```

4. Kubernetes bootstrap (control-plane init + workers join)
```bash
ansible-playbook -i hosts.ini k8s-part3-kubeadm_v2.yml -e @envs/lab.yml --ask-vault-pass
```

5. CNI networking (Calico)
```bash
ansible-playbook -i hosts.ini k8s-part4-calico.yml -e @envs/lab.yml --ask-vault-pass
```

### Option B: Run everything at once (faster when the flow is already validated)

```bash
ansible-playbook -i hosts.ini k8s-all-in-one.yml -e @envs/lab.yml --ask-vault-pass
```

---

## After Deployment

On the control plane node:

```bash
kubectl get nodes
```

You should see all workers in `Ready` state.

---

## Design Principles

- Idempotent execution
- Clear separation of responsibilities
- Minimal manual steps
- Reproducible environments
- Automation-first mindset

---

## Future Improvements

- HA control plane
- External etcd
- Ingress controller
- Observability stack
- GitOps integration

---

## Author

Fernando Avelino
