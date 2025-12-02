# Proxmox VE — Installation and Cluster Setup

This document describes a robust procedure to install Proxmox VE, create a cluster, perform common post-install tasks, and addresses known errors with remediation steps.

# Table of contents
- Overview and goals
- Pre-install checklist
- Install Proxmox VE (ISO)
- Networking and hostname tasks
- Create a new cluster / join nodes
- Post-install tasks (updates, pve-postinstall)
- LVM auto-activation and IOMMU
- Common problems and fixes

# Overview and goals
- Provide a highly-available Proxmox control plane for hosting VMs that will run TrueNAS and Kubernetes nodes.
- Keep cluster quorum healthy and avoid common pitfalls (hostname collisions, leftover configuration files, corosync fingerprint issues).

# Pre-install checklist
- Ensure each node has a static IP on the management network.
- Confirm unique hostnames for every node (no duplicates).
- Time synchronization configured (chrony or systemd-timesyncd).
- BIOS: vt-d / IOMMU enabled if you will passthrough PCI or disks.
- Backup any existing /etc/pve data before re-installing or manipulating clusters.

# Install Proxmox VE
   ## 1. Download ISO: https://www.proxmox.com/en/downloads
   ## 2. Create installation media and boot.
   ## 3. Follow installer:
      - Set a static IP for the management interface.
      - Choose a fully qualified domain name or simple hostname; ensure uniqueness.
   ## 4. After install, update:
      apt update && apt full-upgrade -y
   Consider enabling the enterprise repository only if you have a subscription; otherwise use the no-subscription repository per Proxmox docs.

# Networking and hostname tasks
  ## Set hostname:
     echo "my-proxmox-node" > /etc/hostname
   - Edit /etc/hosts to map all Proxmox node IPs to their hostnames
- Verify: hostnamectl status

# Create a cluster
   ## On the first node (master):
     pvecm create my-cluster
   ## On each additional node:
     pvecm add <IP-of-first-node>
   ## After successful join:
     pvecm status   
     pvecm nodes

# Post-install tasks
- Install community post-install script (optional): https://community-scripts.github.io/ProxmoxVE/scripts?id=post-pve-install
- Enable services you need (pve-cluster, pvedaemon, pveproxy)
- Snapshot a baseline VM configuration after configuration.

# LVM auto-activation after reboot (when VMs cannot see disks)
   ## 1. Check status: vgscan; lvscan
   ## 2. Create systemd unit to activate on boot:
      nano /etc/systemd/system/lvm-activate.service:
        [Unit]
        Description=Auto activate LVM volumes after boot
        DefaultDependencies=no
        After=local-fs.target
        Before=proxmox-backup.service pvedaemon.service pve-storage.service
   
        [Service]
        Type=oneshot
        ExecStart=/sbin/vgchange -ay pve
   
        [Install]
        WantedBy=multi-user.target
   ## 3. Reload Services
         systemctl daemon-reload
         systemctl enable --now lvm-activate.service

# Enable IOMMU and PCI passthrough
   ## Edit /etc/default/grub and set:
     - For Intel:
       GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt"
     - For AMD:
       GRUB_CMDLINE_LINUX_DEFAULT="quiet amd_iommu=on iommu=pt"
   ## Update grub: update-grub && reboot
   ## Verify: dmesg | grep -e DMAR -e IOMMU -e AMD-Vi

# Common problems and fixes

## 1) Join failed due to existing files in /etc/pve/qemu-server
   - Symptom: pvecm add fails with errors mentioning existing VM config files.
   - Cause: leftover qemu-server files conflict with cluster join.
   - Fix:
     ### On the joining node, back up /etc/pve/qemu-server if needed:
          cp -a /etc/pve/qemu-server /root/qemu-server-backup
     ### Remove all files inside /etc/pve/qemu-server (only if they are stale):
          rm -f /etc/pve/qemu-server/*
     - Retry pvecm add <IP>
     - After join, restore configs if appropriate.

## 2) Join failed due to duplicate hostname
- Symptom: pvecm add errors referencing "duplicate name" or cluster refuses node.
- Fix:
  ### Edit /etc/hostname and /etc/hosts to set a unique name:
       nano /etc/hostname
       nano /etc/hosts
  - Reboot the node and retry join.

# 3) Fingerprint not verified / corosync issues
- Symptom: SSH/corosync connection fingerprint errors or corosync refuses nodes.
## Fix: Remove the node from cluster metadata and reinitialize corosync on that node:
     systemctl stop pve-cluster corosync
     pmxcfs -l
     rm -rf /etc/corosync/*
     rm -rf /etc/pve/nodes/*
     rm /etc/pve/corosync.conf || true
     killall pmxcfs || true
     systemctl restart pve-cluster
  - Recreate cluster or re-add node: pvecm add <ip>

# 4) VirtualBox and nested virtualization issues
- Symptom: VM cannot run KVM VMs, cannot use virtualization features
## Fix:
  ### On host:
       sudo rmmod kvm kvm_intel (Intel) or sudo rmmod kvm kvm_amd (AMD)
  - Use bridged adapter in VirtualBox for networking to ensure VMs get an address on your LAN.

# 5) Storage passthrough and TrueNAS visibility in Proxmox
   ## Use /dev/disk/by-id when assigning disks to VMs to ensure stable device references:
      lsblk -o +MODEL,SERIAL
      ls /dev/disk/by-id
  - Example: qm set 101 -scsi1 /dev/disk/by-id/ata-<MODEL>,serial=<SERIAL>

# 6) Cluster quorum problems after node loss
- Symptom: Lost quorum, unable to start VMs on remaining nodes.
- Fix:
  - Follow Proxmox docs to force quorum when necessary:
    pvecm expected 1
  - Re-add missing nodes once they are healthy.

References
- Official Proxmox docs: https://pve.proxmox.com/wiki/Main_Page

If you encounter an issue not documented above, open an issue in this repository and include outputs for pvecm status, systemctl status pve-cluster, and relevant logs.
