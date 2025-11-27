# Proxmox Cluster with Kubernetes — Installer Guide

This repository contains a professionally written, step‑by‑step installer and troubleshooting guide to build a small lab or production-ready environment that combines:

- Proxmox VE (virtualization and cluster management)
- TrueNAS (storage backing — pools, ZFS)
- Kubernetes (container orchestration)

This top-level README links to three focused guides that walk through installation, configuration, common pitfalls and fixes:

- docs/PROXMOX_SETUP.md — Proxmox VE installation, cluster creation, post-install tuning, and common fixes.
- docs/TRUENAS_SETUP.md — TrueNAS deployment as a VM/physical host, disk passthrough, pool creation and troubleshooting.
- docs/KUBERNETES_SETUP.md — Kubernetes deployment options (kubeadm / k3s), networking (CNI), storage integration (TrueNAS CSI/NFS), common errors and resolutions.

If you are here to follow a full end-to-end setup, read the files in that order: Proxmox → TrueNAS → Kubernetes.

Quick links
- Proxmox guide: ./docs/PROXMOX_SETUP.md
- TrueNAS guide: ./docs/TRUENAS_SETUP.md
- Kubernetes guide: ./docs/KUBERNETES_SETUP.md

Why follow this guide
- Consolidates best practices for Proxmox + TrueNAS + Kubernetes.
- Documents real-world problems encountered during installs and their fixes.
- Gives reproducible commands, configuration snippets, and checklists.

Recommended prerequisites
- Hardware: 3+ physical nodes recommended for highly available Proxmox cluster (minimum 2 for testing). Each node: 8+ vCPUs, 32+ GB RAM, fast disks for VMs and at least one disk per TrueNAS pool member.
- Network: VLAN-capable switch preferred. Static IPs for Proxmox nodes and TrueNAS; management LAN reachable from your workstation.
- Software: Proxmox VE 7.x / 8.x (follow official compatibility notes), TrueNAS SCALE/CORE (depending on ZFS needs), recent Linux for Kubernetes nodes (Debian/Ubuntu 22.04+ recommended).
- Basic Linux, storage (ZFS/LVM) and networking knowledge.

Repository structure
- README.md (this file)
- docs/PROXMOX_SETUP.md
- docs/TRUENAS_SETUP.md
- docs/KUBERNETES_SETUP.md

Support and issues
- Open issues in this repository for any incorrect instructions or missing steps.
- When reporting a bug include:
  - Which guide you were following (file and section)
  - Proxmox, TrueNAS, Kubernetes versions
  - Relevant logs or command outputs (anonymize sensitive data)

Contributing
- Corrections, additional troubleshooting steps, or alternative deployment approaches are welcome — open a PR with the proposed change.

License
- See LICENSE (if present) or assume permissive use for documentation. Confirm before using in a commercial distribution.

Notes and disclaimers
- This guide consolidates community knowledge and personal lab experience. Always validate steps in a test environment before production rollouts.
- The instructions include commands that operate on storage, disks, and ZFS. Take backups before running destructive commands.

Next steps
- Open docs/PROXMOX_SETUP.md and follow the Proxmox installation and cluster steps.
