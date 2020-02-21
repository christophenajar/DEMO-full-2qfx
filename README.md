
# DEMO-full-2qfx

This Vagrantfile will spawn 2 instances of VQFX (Full) each with 1 Routing Engine and 1 PFE VM  
Both VQFX will be connected back to back with IP address pre-configured on their interfaces

# Requirement

### Resources
 - RAM : 5G
 - CPU : 3 Cores

### Tools
 - Ansible for provisioning (except for windows)
 - Junos module for Ansible

# Topology

        em0|                        em0|
    =============  xe-0/0/[0-5] =============
    |           | ------------- |           |
    |   vqfx1   | ------------- |   vqfx2   |
    |           | ------------- |           |
    =============               =============
        em1|                        em1|
    =============               =============
    | vqfx1-pfe |               | vqfx1-pfe |
    =============               =============

# Provisioning / Configuration

Ansible is used to preconfigured both VQFX with an IP address on their interfaces

**1) Install Juniper vqfx10k boxes**

`vagrant box add juniper/vqfx10k-re`

`vagrant box add juniper/vqfx10k-pfe`


**2) Start pipenv**

`pipenv shell`


**3) Install python packages**

`pip install -r requirements.txt`


**4) start vagrant**

`vagrant up`

