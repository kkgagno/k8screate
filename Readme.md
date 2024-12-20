# Linux Patching Playbook (Ubuntu/Debian/RHEL/CentOS/Amazon Linux)
Prerequirements:
Ubuntu server to run on
    -with Google SDK installed (there by default if GCP image)  
    -Ansible installed  (apt-get update),(apt-get install ansible)   
    -Ansible galaxy collection GCE snap module (ansible-galaxy collection install community.google)  
    -Authed to GCP SDK with proper permissions osadmin, ...etc  
    -(Compute Instance Admin or custom) permissions needed from service account being used by ansible host machine/service account authed  
            - If Custom role, the following permissions are needed:  
            compute.disks.createSnapshot  
            compute.disks.get  
            compute.disks.list  
            compute.instances.get  
            compute.instances.list  
            compute.regions.list  
            compute.snapshots.create  
            compute.snapshots.get  
            compute.snapshots.list  
            compute.zoneOperations.get  
            compute.zones.get  
            compute.zones.list   
Private key used for ssh in secret manager
Service account used for snapshots JSON stored in secret manager


To note: keep the "devbox" hosts group in there for the reason of storing private key/svc acct json within.  It is skipped in the code when  
"(when:  (inventory_hostname != "localhost") )" is in place.  Key is needed for SSH'ing to instances.  Service account json needed for snapshots.  

### How to run:
```
###Configure vars.yml

Example contents below:  (Fill in the project name which contains your secrets, your private key secret name,  
svc acct json secret name, and svc acct name **should not need to change svc acct json or svc acct name)  

#Project containing secrets
gcpproject: test-ground
#secret name for private key used for ssh
sshcred: sshcred
#secret name for Service Account JSON
scjson: gitmanjson
svcacct: gitman@test-ground.iam.gserviceaccount.com

###Download this repo and change directory into it.

###To test connectivity, (this will show OS version as well as last 20 lines of unattended dryrun)
    - ansible-playbook test.yaml -i hosts -l preprodgroup1:devbox

###To create snapshots, run the playbook snapsonly.yaml
    - ansible-playbook snapsonly.yaml -i hosts -l preprodgroup1:devbox

###To run the security updates, run the playbook runupdates.yaml
    - ansible-playbook runupdates.yaml -i hosts -l preprodgroup1:devbox

###To add additional groups, edit the hosts file in your local repo and add.  Use preprodgroup1 as an example.
```

The playbook does reboot hosts by default.  


- to make work with gcp.yml file as inventory - specify projects and will detect all and run  
- using gcp.yml enables ansible to grab GCP related info on instances and can be pulled into a yaml file  
- this would work nicely if we were patching full list of hosts in projects.  
- Steps to be able run:  
sudo apt install python3-pip  
sudo pip3 install google-auth  
ex.. ansible-playbook snapsonly.yaml -i ansible.gcp.yml  

---

 
