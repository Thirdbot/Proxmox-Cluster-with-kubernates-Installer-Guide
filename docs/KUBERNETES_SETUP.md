# Kubernetes — Deployment and Storage Integration

This document covers recommended Kubernetes deployment strategies on top of Proxmox VMs with TrueNAS providing persistent storage. It provides step-by-step options (kubeadm for upstream k8s or k3s for lightweight), networking/CNI guidance, storage integration, and common troubleshooting.
# first install debian
  - click https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-13.2.0-amd64-netinst.iso
  - create vm in proxmox
  - install update and upgrade pacakage
  - install cifs-utils (for smb share)
  - create directory /mnt/prox-share
  - sudo nano /etc/fstab (to make endpoint)
  - //{your smb endpoint} {mount directory} cifs credentials=/root/smbcredentials uid={your user id} sid={your user id} 0 0
  - nano /root/smbcredentials
    user={your user}
    password={your password}
    save and exits
    
Which Kubernetes distribution to use?
- kubeadm (upstream Kubernetes): full control, production-grade, more configuration.
- k3s (Rancher): lightweight, simpler for labs and edge clusters.
- Choose based on resource constraints and team experience.

Common prerequisites
- Disable swap on all nodes: swapoff -a and remove swap entries from /etc/fstab.
- Ensure container runtime installed (containerd recommended).
- Ensure correct cgroup driver: containerd + systemd cgroup driver is common for kubeadm.
- Open required ports between control plane and worker nodes:
  - 6443 (kube-apiserver), etcd ports (2379-2380) for stacked etcd, NodePort ranges, CNI ports.

Option A — kubeadm (recommended for production-style clusters)

1) Prepare each node
- Install containerd:
  - apt-get install -y containerd
  - Configure /etc/containerd/config.toml with systemd cgroup:
    systemd_cgroup = true
  - systemctl restart containerd
- Install kubeadm, kubelet, kubectl (matching versions):
  - apt-get update && apt-get install -y kubelet kubeadm kubectl
- Disable swap: swapoff -a

2) Initialize control plane (first master)
- kubeadm init --pod-network-cidr=10.244.0.0/16
- Save the kubeadm join command printed at the end for workers.

3) Configure kubectl for admin user:
- mkdir -p $HOME/.kube
- sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
- sudo chown $(id -u):$(id -g) $HOME/.kube/config

4) Install a CNI (e.g., Calico or Flannel)
- Example Flannel:
  - kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml

5) Join workers
- On worker nodes: kubeadm join <control-plane-ip>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>

Option B — k3s (faster for labs)
- Install: curl -sfL https://get.k3s.io | sh -
- For HA k3s, follow the k3s server + agent join token procedure.

Storage provisioning (using TrueNAS)
- Two principal approaches:
  1. NFS: Export a directory/dataset from TrueNAS; create a StorageClass that provisions PersistentVolumes using dynamic provisioning via an NFS provisioner.
  2. CSI: Use a TrueNAS CSI driver (or community driver) to provision ZFS datasets as PVs dynamically (recommended for feature parity).

Example: Using NFS (quick start)
- Create NFS export on TrueNAS for k8s.
- On Kubernetes cluster:
  - Install NFS Client Provisioner or standard provisioner via Helm, configured to point to the TrueNAS NFS server.

Example: Using CSI (preferred)
- Follow the TrueNAS CSI driver installation docs.
- Create StorageClass that uses the CSI driver for dynamic volumes.

Common Kubernetes problems and fixes

1) kubeadm join fails (certificate or token issues)
- Symptom: "x509: certificate signed by unknown authority" or "token expired"
- Fix:
  - Ensure the token is current (kubeadm token create --print-join-command on control plane).
  - Ensure network connectivity and that the control plane IP in the join command is reachable.
  - Check system time synchronization (NTP/chrony).

2) kubelet not ready / Image pull errors
- Symptom: kubelet status shows CrashLoopBackOff or NotReady.
- Fix:
  - Check journalctl -u kubelet
  - Verify container runtime is running: systemctl status containerd
  - Ensure correct cgroup driver: mismatch between kubelet and containerd requires aligning config.

3) CNI plugin causes pods to remain pending
- Symptom: pods show Pending and no CNI network attachments.
- Fix:
  - Ensure the CNI manifest was applied after kubeadm init.
  - Check kubectl get pods -n kube-system and examine CNI pod logs.

4) PersistentVolume provisioning fails
- Symptom: PVC stuck in Pending with no provisioner.
- Fix:
  - Ensure a StorageClass exists and a matching provisioner is deployed.
  - For NFS: ensure network connectivity and permissions from Kubernetes nodes to TrueNAS NFS export.
  - For CSI: check driver pods in kube-system namespace and driver logs.

5) DNS resolution (CoreDNS) issues
- Symptom: cluster DNS failing, pods cannot resolve names.
- Fix:
  - kubectl -n kube-system get pods -l k8s-app=kube-dns
  - Check CoreDNS logs, ensure the CNI hasn't blocked required networking.

6) Control plane availability and etcd problems
- Symptom: etcd failing to form quorum on multi-master kubeadm setups.
- Fix:
  - Verify etcd member list and endpoints.
  - Restore from a snapshot if needed, or recreate control-plane nodes carefully.

Useful debugging commands
- kubectl get nodes
- kubectl get pods --all-namespaces
- kubectl describe pod <pod> -n <ns>
- journalctl -u kubelet -b
- systemctl status containerd

Deploying common tooling
- ArgoCD (GitOps): Install in a dedicated namespace and connect to your git repo.
- Portainer: Install for cluster/VM/container management if desired.
- Monitoring: Prometheus + Grafana for cluster and node metrics.

Security
- Always use RBAC for production clusters.
- Keep Kubernetes and container runtime up-to-date.
- Limit access to kubeconfig files and the Proxmox GUI.

References
- Kubernetes docs: https://kubernetes.io/docs/
- kubeadm: https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
- k3s: https://k3s.io/
