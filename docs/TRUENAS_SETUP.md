# TrueNAS — Installation and Storage Integration

This guide focuses on deploying TrueNAS (SCALE or CORE) as a VM or physical host to provide persistent storage for your Proxmox/Kubernetes environment. It includes disk passthrough, pool creation, and troubleshooting.

Important: TrueNAS manages disks directly. When passing disks to TrueNAS, be careful: the disks will be consumed by ZFS. Do not pass disks with data you need to keep, unless you have backups.

Preparation
- Choose TrueNAS SCALE (Linux-based) vs CORE (FreeBSD) according to your needs. SCALE has Linux container/VM friendly toolchains.
- Ensure nodes intended for TrueNAS have exclusive disks available for ZFS.
- If using a VM, prefer direct disk passthrough via PCI (HBA) or /dev/disk/by-id paths.

Create a TrueNAS VM in Proxmox
1. Download the TrueNAS ISO from https://www.truenas.com/download-truenas-core/ or SCALE page.
2. Create a VM:
   - OS: Other or Linux (TrueNAS will boot from ISO)
   - CPU type: kvm64 (or host for near-native behavior)
   - BIOS: OVMF if you need UEFI, otherwise default SeaBIOS works
   - Disk: leave small OS disk for TrueNAS installer (if installing to a virtual disk)
   - Do NOT add the storage pool disks as default Proxmox disks — pass them through below.
3. Passthrough disks:
   - Identify disks: lsblk -o NAME,SIZE,MODEL,SERIAL
   - Prefer /dev/disk/by-id references:
     ls /dev/disk/by-id
   - Assign to VM:
     qm set <vmid> -scsi1 /dev/disk/by-id/ata-<MODEL>-serial=<SERIAL>,serial=<SERIAL>
   - For many-disk pools, consider using PCI HBA passthrough for full disk access and performance.

Install TrueNAS
- Boot the VM from the ISO and follow install steps.
- Create the admin user and set a secure password (default root for CORE; check SCALE docs).
- After install, access the web UI and configure networking, NTP, and SSH.

Create a ZFS pool and datasets
- In the GUI, Storage → Pools → Add
- Select disks (use correct topology: RAIDZ, mirror, or stripe depending on redundancy needs)
- Create datasets and configure permissions for Kubernetes storage classes.
Mount folder
   New-PSDrive -Name Z -PSProvider FileSystem -Root "\\10.0.23.128\truenas_nfs" -Persist 
Topology errors when adding disks
- Symptom: “Topology error” or disk not writable
- Causes & fixes:
  - Disk is still in use by Proxmox (e.g., part of LVM or has partitions).
    - Clear partitions: wipefs -a /dev/sdX (careful: destructive)
    - Alternatively, use sgdisk --zap-all /dev/sdX
  - Disk device path changed (use by-id to avoid this).
  - Disk uses RAID metadata from previous usage. Clear metadata with:
    mdadm --zero-superblock /dev/sdX
    wipefs -a /dev/sdX

Example: Assign a disk to a VM using by-id
- Identify:
  ls /dev/disk/by-id | grep ata
- Assign:
  qm set 101 -scsi1 /dev/disk/by-id/ata-ST2000LM015-2E8174_ZDZRJEXN,serial=ZDZRJEXN

Common issues and fixes

1) TrueNAS cannot see disks passed from Proxmox
- Verify the VM has direct access to the device (not a partition).
- Confirm /dev/disk/by-id inside the VM.
- If passthrough via HBA, ensure IOMMU is enabled and HBA is assigned to the VM.

2) Pool import errors after attach to another system
- Use zpool import -f <poolname>
- If the pool GUID is found but shows errors, export from the previous host before attach:
  zpool export <poolname>

3) ZFS performance issues
- Avoid thin provisioning for TrueNAS pool disks.
- Use appropriate recordsize and compression settings for the workload.
- Monitor ARC and tune if necessary.

Integrating TrueNAS with Kubernetes
- Options:
  - TrueNAS CSI driver (official or community-supported).
  - NFS exports (easy to set up, less featureful).
  - iSCSI for block storage (useful for specific workloads).
- For dynamic provisioning, prefer a CSI driver that supports ZFS datasets.

Security and maintenance
- Secure API/UI endpoints with strong passwords and network restrictions.
- Regularly update TrueNAS to keep security patches applied.
- Monitor SMART for disks: in TrueNAS GUI, enable SMART tests.

References
- TrueNAS docs: https://www.truenas.com/docs/
- TrueNAS SCALE vs CORE considerations
