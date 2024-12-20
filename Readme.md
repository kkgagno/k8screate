# Linux Patching Playbook (Ubuntu/Debian/RHEL/CentOS/Amazon Linux)
Prerequirements:
- Ubuntu 22.04 server for all components, ssh keypair usage required
- Development Box, Ubuntu 22.04, Ansible 2.17.7, (MySQL 8.0.40 required for Semaphore)
      #Install pipx AND ansible
       sudo apt install pipx
       pipx install --include-deps ansible
       pipx ensurepath
  
- MySql 8 on Devbox
       apt install mysql-server
          run, mysql, create user for semaphore and give permission ..ex:
              CREATE USER 'keith'@'localhost' IDENTIFIED BY 'password';
              GRANT ALL PRIVILEGES ON *.* TO 'keith'@'localhost';
  
- On Devbox also Download semaphore manually  
        wget https://github.com/semaphoreui/semaphore/releases/download/v2.10.43/semaphore_2.10.43_linux_amd64.tar.gz
        tar -xvf semaphore_2.10.43_linux_amd64.tar.gz
        running setup creates the json
        ./semaphore setup
        ./semaphore server --config /root/config.json &
  



    

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

 
