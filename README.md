# Proxmox-Cluster-with-kubernates-Installer-Guide
## This guide made for set up a Proxmox to run clusters of kubernates services for workflow
## Application
    1.ArgoCD for gitops ci/cd through git repository
    2.pulse proxmox for proxmox cluster configuration and monitoring
    3.Portainer a container management for kubernates
    4.homepage for monitoring all application (All above)
### kubernates set up from this tutorial.
    https://www.youtube.com/watch?v=mNp_9iMIqH0&list=LL&index=6
### Set up Clusters
#### if you dont have a router avalible. pls use a bridge adapter to convert wifi to ethernet. then access it to same network.
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
## 3. using virtual box on ubuntu
    ### cant operate on vmx root mode
        sudo rmmod kvm kvm_intel #if intel
        sudo rmmod kvm_amd kvm #if amd
    ### cant connect to ip
        choose bridge network in virtual box
        check ip to be 192.168.x.x

##After Installation
    ### Install pve post installer from
        https://community-scripts.github.io/ProxmoxVE/scripts?id=post-pve-install
        with shell setting things up and then click upgrade in repository pages.

#Install Truenas
## 1. Download or Upload Truenas
## 2. Create a vm with cpu type kvm64
## 3. Setup Truenas

        
