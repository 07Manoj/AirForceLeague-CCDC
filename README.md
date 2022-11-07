# AirForceLeague-CCDC

This repository contains the configuration files for the Air Force Cyber Defense League organized by University of Massachusetts Lowell  Research Institute.  

The task is to deploy infrastructure using an automation tool called  "ansible" and perform specific tasks which would secure the air force infrastructure.

Follow the below mentioned steps to deploy the infrastructure on any machine.

Consider a production environment where you have multiple servers. You will be using a single computer / workstation to provision and manage all the servers. 

I am using a docker container running ubuntu as my workstation. I am a root user in the container. 

Part 1:  Install ansible and community.docker on the workstation. The ansible galaxy community.docker contains many plugins that will help you use docker with ansible.
```
apt install ansible
ansible-galaxy collection install community.docker
```

