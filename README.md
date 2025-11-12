# Proxmox-Cluster-with-kubernates-Installer-Guide
## This guide made for set up a Proxmox to run clusters of kubernates services for workflow

### Set up Clusters
    1. Install proxmox via proxmox installer here-> https://www.proxmox.com/en/products/proxmox-virtual-environment/get-started on machine or virtual machine.
    2. After installation done. Set up a credential.
    3. Open up machines IP and Login.
    4. Select One of machines to be creating Cluster
    5. Copy Join information and apply it to other machines.
    6. After Sucessful Connecting, node members appear in cluster group.
    7. Set up Cluster Compleated
    8. Refresh web pages and Re-Login


#Problem that might be occurs.
## 1. join failed due to existed files or same node's name
  ### 1.1 exited files
        1. Go to datacenter's pve and select shell to get into console or ssh into with an ip
        2. Get to /etc/pve/qemu-server and delete files or copy it to external source (as long as there is nothing in qemu-server) later, copy it back to same place /etc/pve/qemu-server
        3. Re-join the cluster using join information
  ### 1.2 same node's name
        1. Go to datacenter's pve and select shell to get into console or ssh into with an ip
        2. nano /etc/hosts and nano /etc/hostname then change the name you see into name you prefer then save
        3. reboot
        4. refresh webpages and Re-Login

## 2. finger printed is not verified
  ### 2.1 remove Cluster Node
        systemctl stop pve-cluster corosync
        pmxcfs -l
        rm -fR /etc/corosync/*
        rm -fR /etc/pve/nodes/*
        rm /etc/pve/corosync.conf
        killall pmxcfs
        systemctl restart pve-cluster
  ### 2.2 recreating a cluster or adding nodes
        follow Set up Cluster or pvecm add ip.ip.ip.1
