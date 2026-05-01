# K3s Cluster Configuration Guide

This guide provides a comprehensive step-by-step walkthrough for setting up a 3-node K3s cluster on VMware VMs with static IP addresses and remote access configuration.

---

## Table of Contents

1. [Network Configuration (Static IPs)](#1-network-configuration-static-ips)
2. [K3s Cluster Installation](#2-k3s-cluster-installation)
3. [Remote Access Configuration](#3-remote-access-configuration)

---

## 1. Network Configuration (Static IPs)

Perform these steps to ensure your VMs have persistent, static IP addresses for both internet access and cluster communication.

### 1.1 Disable Cloud-Init Network Management
**Run on all 3 VMs:**

1. Create/edit the configuration file:
   ```bash
   sudo nano /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg
   ```
2. Add the following line:
   ```yaml
   network: {config: disabled}
   ```
3. Save and exit (`Ctrl+X` → `Y` → `Enter`).

### 1.2 Configure Netplan
**Run on all 3 VMs:**

Edit the existing Netplan file:
```bash
sudo nano /etc/netplan/50-cloud-init.yaml
```

Replace the entire content with the configuration corresponding to the VM:

#### On k3s-master
```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    ens33:
      dhcp4: no
      addresses: [192.168.88.10/24]
      routes:
        - to: default
          via: 192.168.88.2
      nameservers:
        addresses: [8.8.8.8, 1.1.1.1]
    ens36:
      dhcp4: no
      addresses: [192.168.66.10/24]
```

#### On k3s-worker1
```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    ens33:
      dhcp4: no
      addresses: [192.168.88.11/24]
      routes:
        - to: default
          via: 192.168.88.2
      nameservers:
        addresses: [8.8.8.8, 1.1.1.1]
    ens36:
      dhcp4: no
      addresses: [192.168.66.11/24]
```

#### On k3s-worker2
```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    ens33:
      dhcp4: no
      addresses: [192.168.88.12/24]
      routes:
        - to: default
          via: 192.168.88.2
      nameservers:
        addresses: [8.8.8.8, 1.1.1.1]
    ens36:
      dhcp4: no
      addresses: [192.168.66.12/24]
```

### 1.3 Apply Network Changes
**Run on all 3 VMs:**

1. Fix file permissions:
   ```bash
   sudo chmod 600 /etc/netplan/50-cloud-init.yaml
   ```
2. Test the configuration (dry run):
   ```bash
   sudo netplan try
   ```
   *If successful, press `Enter` within 120s to confirm.*
3. Apply permanently:
   ```bash
   sudo netplan apply
   ```

### 1.4 Verification
Verify interfaces and connectivity:
```bash
# Check IPs
ip a

# Test Internet
ping -c 3 8.8.8.8

# Test Cluster Network (from Master)
ping -c 3 192.168.66.11
ping -c 3 192.168.66.12
```

---

## 2. K3s Cluster Installation

### 2.1 Environment Preparation
**Run on all 3 VMs:**

> [!IMPORTANT]
> Kubernetes requires swap to be disabled for stability.

1. Disable firewall:
   ```bash
   sudo ufw disable
   ```
2. Disable swap:
   ```bash
   sudo swapoff -a
   sudo sed -i '/ swap / s/^/#/' /etc/fstab
   ```
3. Set hostnames (if not already set):
   - **Master:** `sudo hostnamectl set-hostname k3s-master`
   - **Worker 1:** `sudo hostnamectl set-hostname k3s-worker1`
   - **Worker 2:** `sudo hostnamectl set-hostname k3s-worker2`
4. Update `/etc/hosts`:
   ```bash
   sudo nano /etc/hosts
   ```
   Add these lines:
   ```text
   192.168.66.10  k3s-master
   192.168.66.11  k3s-worker1
   192.168.66.12  k3s-worker2
   ```

### 2.2 Master Node Installation
**Run on k3s-master:**

```bash
curl -sfL https://get.k3s.io | sh -s - \
  --node-ip 192.168.66.10 \
  --advertise-address 192.168.66.10 \
  --flannel-iface ens36 \
  --write-kubeconfig-mode 644
```

Verify the installation:
```bash
sudo systemctl status k3s
sudo k3s kubectl get nodes
```

### 2.3 Join Worker Nodes
1. **Get the token from Master:**
   ```bash
   sudo cat /var/lib/rancher/k3s/server/node-token
   ```
   *Copy this token.*

2. **Join Worker 1:**
   ```bash
   curl -sfL https://get.k3s.io | K3S_URL=https://192.168.66.10:6443 \
     K3S_TOKEN=<TOKEN> \
     sh -s - agent \
     --node-ip 192.168.66.11 \
     --flannel-iface ens36
   ```

3. **Join Worker 2:**
   ```bash
   curl -sfL https://get.k3s.io | K3S_URL=https://192.168.66.10:6443 \
     K3S_TOKEN=<TOKEN> \
     sh -s - agent \
     --node-ip 192.168.66.12 \
     --flannel-iface ens36
   ```

### 2.4 Cluster Verification
**Run on Master:**
```bash
sudo k3s kubectl get nodes -o wide
```
Check that all nodes are `Ready` and use `192.168.66.x` as their `INTERNAL-IP`.

### 2.5 Deployment Test
**Run on Master:**

Deploy a simple Nginx service to confirm the cluster works end-to-end:
```bash
sudo k3s kubectl create deployment nginx --image=nginx --replicas=2
sudo k3s kubectl get pods -o wide
```
Verify that pods are scheduled across different worker nodes.

---

## 3. Remote Access Configuration

Configure your host machine to manage the cluster using `kubectl`.

### 3.1 Copy Kubeconfig to Host
**Run on your host machine:**

- **Linux/Mac:**
  ```bash
  scp houssem@192.168.66.10:/etc/rancher/k3s/k3s.yaml ~/.kube/config
  ```
- **Windows (PowerShell):**
  ```powershell
  scp houssem@192.168.66.10:/etc/rancher/k3s/k3s.yaml $HOME\.kube\config
  ```

### 3.2 Update Server Address
The default config points to `127.0.0.1`. Update it to the Master's IP.

- **Linux/Mac:**
  ```bash
  sed -i 's/127.0.0.1/192.168.66.10/g' ~/.kube/config
  ```
- **Windows (PowerShell):**
  ```powershell
  (Get-Content $HOME\.kube\config) -replace '127.0.0.1', '192.168.66.10' | Set-Content $HOME\.kube\config
  ```

### 3.3 Test Connectivity
**Run on your host machine:**
```bash
kubectl get nodes -o wide
```
You should now see the cluster status directly from your host.
