
# Ansible Kubernetes Jarvice deployment
```
Prerequirements:
- Ubuntu 22.04 server for all components, ssh keypair usage required

Devbox:
- Development Box, Ubuntu 22.04, Ansible 2.17.7, (MySQL 8.0.40 required for Semaphore)
      #Install pipx AND ansible procedure
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
  
```


    


###  Deploying Kubernetes Cluster with Ansible
```
###Configure k8svars.yml

Example contents below: specify your pod network, LB endpoint, metallb service IP range, and each of the 3 control planes.   

#Pod network applied with Cilium
podnetwork: 10.244.0.0/16
#K8s Endpoint nginx LB
nginxlb: 192.168.122.132
#Metallb service IP range for services
metallb: 192.168.122.240-192.168.122.250
#control plane1, this is too populate config file for nginx LB
controlplane1: 192.168.122.252
#control plane2, this is too populate config file for nginx LB
controlplane2: 192.168.122.161
#control plane3, this is too populate config file for nginx LB
controlplane3: 192.168.122.23

Manual way from devbox without Semaphore;
###Download this repo and change directory into it.
k8shosts needs to be populated as well as k8svars.yaml.

###Full run, creates endpoint, installs components to controllers and workers, creates 3 control planes and workers listed in k8shosts file.  Sections can be commented out of of yaml if needing to troubleshoot OR each playbook can be ran seperately outside of k8screate.yaml.  Cat file to see books imported.
    - ansible-playbook k8screate.yaml -i k8shosts

##TO RUN EACH SEPERATELY, EACH OF THESE IS CALLED IN BELOW ORDER WITHIN k8screate.yaml. k8shosts needs to be populated as well as k8svars.yaml.
###To create NGINX load balancer for Endpoint
    - ansible-playbook k8s_endpoint.yaml -i k8shosts

###To install requirements onto main controller
    - ansible-playbook k8s_ctrl_reqs.yaml -i k8shosts

###To install requirements onto other 2 controllers
    - ansible-playbook k8s_addl_ctrl_reqs.yaml -i k8shosts

###To install requirements onto worker nodes
    - ansible-playbook k8s_wrkr_reqs.yaml -i k8shosts

###To install requirements onto worker nodes
    - ansible-playbook k8s_wrkr_reqs.yaml -i k8shosts

###To initiate first controller and get info for additonal controllers and worker join info
    - ansible-playbook k8s_controller.yaml -i k8shosts

###To Add second and third control planes
    - ansible-playbook k8s_addl_controllers.yaml -i k8shosts

###To Add worker nodes to cluster
    - ansible-playbook k8s_workers.yaml -i k8shosts

###To Add CIS lockdown settings to Ubuntu 22
    - ansible-playbook ./UBUNTU22-CIS/site.yml -i k8shosts

###To install metallb into cluster
    - ansible-playbook k8s_metallb.yaml -i k8shosts



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

 
