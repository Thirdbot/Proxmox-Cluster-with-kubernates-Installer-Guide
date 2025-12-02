# Terraform Setup for autommate resources in proxmox

## Create User for Terraform in Proxmox in machine's shell
  ### add user
    sudo pveum user add terraform@pve
  ### add role
    pveum role add Terraform -privs "Datastore.AllocateSpace Datastore.AllocateTemplate Datastore.Audit Pool.Allocate Pool.Audit Sys.Audit Sys.Console Sys.Modify VM.Allocate VM.Audit VM.Clone VM.Config.CDROM   VM.Config.Cloudinit VM.Config.CPU VM.Config.Disk VM.Config.HWType VM.Config.Memory VM.Config.Network VM.Config.Options VM.Migrate VM.PowerMgmt SDN.Use"
  ### assign role
    pveum aclmod / -user terraform@pve -role Terraform
  
    

  
