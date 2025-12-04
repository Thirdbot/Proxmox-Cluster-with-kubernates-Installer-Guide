# Terraform Setup for autommate resources in proxmox https://www.youtube.com/watch?v=5sI2N6Klji4

## Create User for Terraform in Proxmox in machine's shell
  ### add user
    sudo pveum user add terraform@pve
  ### add role
    pveum role add Terraform -privs "Datastore.AllocateSpace Datastore.AllocateTemplate Datastore.Audit Pool.Allocate Pool.Audit Sys.Audit Sys.Console Sys.Modify VM.Allocate VM.Audit VM.Clone VM.Config.CDROM   VM.Config.Cloudinit VM.Config.CPU VM.Config.Disk VM.Config.HWType VM.Config.Memory VM.Config.Network VM.Config.Options VM.Migrate VM.PowerMgmt SDN.Use"
  ### assign role
    pveum aclmod / -user terraform@pve -role Terraform
  ### create api token
    pveum user token add terraform@pve provider --privsep=0

## set up terraform with this template from https://github.com/Thirdbot/proxmox-terraform
  ## create ssh key gen
    #in powershell
    ssh-keygen
  ## check key activation
    #in powershell
    ssh-add -L
  ## auto activate key
    #in powershell
    Set-Service ssh-agent -StartupType Automatic
    Start-Service ssh-agent
  ## set root password (the same password for login as root in proxmox)
      #in powershell
      $env:TF_VAR_proxmox_password="example password"
  ## configure file variables.
    # after that run
      terraform validate
      terraform apply
      
    

  
