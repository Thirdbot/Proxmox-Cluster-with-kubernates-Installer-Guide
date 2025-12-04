# Proxmox Cluster with Kubernetes — Installer Guide

This repository contains a professionally written, step‑by‑step installer and troubleshooting guide to build a small lab or production-ready environment that combines:

- Proxmox VE (virtualization and cluster management)
- TrueNAS (storage backing — pools, ZFS)
- Kubernetes (container orchestration)

This top-level README links to three focused guides that walk through installation, configuration, common pitfalls and fixes:

- docs/PROXMOX_SETUP.md — Proxmox VE installation, cluster creation, post-install tuning, and common fixes.
- docs/TRUENAS_SETUP.md — TrueNAS deployment as a VM/physical host, disk passthrough, pool creation and troubleshooting.
- docs/PORTAINER(DOCKER)_SETUP.md — DOCKER deployment options , storage integration (TrueNAS CSI/NFS), common errors and resolutions.

Quick links
- Proxmox guide: ./docs/PROXMOX_SETUP.md
- TrueNAS guide: ./docs/TRUENAS_SETUP.md
- DOCKER guide: ./docs/PORTAINER(DOCKER)_SETUP.md
