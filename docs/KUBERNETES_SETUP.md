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

# Docker Installation as lxc
  - bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/docker.sh)" in machine's shell
  - seperate container and template storage location (template in nfs and container in local lvm)
  - done
# Portainer Installation using docker
  -in Docker Installation add option Installing portainer as GUI
  or follow this https://docs.portainer.io/start/install-ce/server/docker/linux

# Potential Problem
  - Since this run in lxc.There is a Permission Problem ex. sysctl net.ipv4.ip_unprivileged_port_start file: reopen fd 8: permission denied
  - fixing vm config nano /etc/pve/lxc/<CTID>.conf in machine's shell
  - add
  - lxc.apparmor.profile: unconfined
  - lxc.cgroup2.devices.allow: a
  - lxc.cap.drop:
  - save and exit
  - reboot vm
  - try docker run hello-word
  

Deploying common tooling
- ArgoCD (GitOps): Install in a dedicated namespace and connect to your git repo.
- Portainer: Install for cluster/VM/container management if desired.
- Monitoring: Prometheus + Grafana for cluster and node metrics.
