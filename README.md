## What is Infrastructure as code (IaC)
* Helps codify everything
* Helps us build and delete things
* 
* Infrastructure as code (IaC) uses DevOps methodology and versioning with a descriptive model to define and deploy infrastructure, such as networks, virtual machines, load balancers, and connection topologies. Just as the same source code always generates the same binary, an IaC model generates the same environment every time it deploys
* Ansible configuration management
* Terraform orchestration
* IaC evolved to solve the problem of environment drift in release pipelines. Without IaC, teams must maintain deployment environment settings individually. Over time, each environment becomes a "snowflake," a unique configuration that can't be reproduced automatically
*  Inconsistency among environments can cause deployment issues. Infrastructure administration and maintenance involve manual processes that are error prone and hard to track
* IaC avoids manual configuration and enforces consistency by representing desired environment states via well-documented code in formats such as JSON
* Infrastructure deployments with IaC are repeatable and prevent runtime issues caused by configuration drift or missing dependencies. Release pipelines execute the environment descriptions and version configuration models to configure target environments. To make changes, the team edits the source, not the target
* Idempotence, the ability of a given operation to always produce the same result, is an important IaC principle
* A deployment command always sets the target environment into the same configuration, regardless of the environment's starting state. Idempotency is achieved by either automatically configuring the existing target, or by discarding the existing target and recreating a fresh environment.
## Configuration management
* Configuration Management is the process of maintaining systems, such as computer hardware and software, in a desired state.
## Orchestration
* Orchestration is the automated configuration, management, and coordination of computer systems, applications, and services
* Terraform.
* Ansible.
* AWS CloudFormation.
* Azure Resource Manager.
* Google Cloud Deployment Manager.
* Chef.
* Puppet.
* SaltStack.
## Terraform
* Terraform is an infrastructure as code tool that lets you build, change, and version cloud and on-prem resources safely and efficiently
* Not cloud dependent, can use with any cloud provider, same as ansible
## Ansible
* Simple and powerful
* Agentless -> dont need ansible installed in every agent node
* Communicates with SSH
* Uses python
* 
* Implement configuration management for IaC
* Uses YAMAL as a language
* Ansible is the simplest way to automate apps and IT infrastructure. Application Deployment + Configuration Management + Continuous Delivery.
* Ansible can be used to provision the underlying infrastructure of your environment, virtualized hosts and hypervisors, network devices, and bare metal servers. It can also install services, add compute hosts, and provision resources, services, and applications inside of your cloud
## Push pull configuration
* Pull Model: The nodes are dynamically updated with the configurations that are present in the server
* Push Model: Centralized server pushes the configurations on the nodes
## Who is using IaC
* Home office
* Sky
* Banking
* sudo ansible -m ping web/all

![](images/ansible_structure.png)

## Setting up 3 vms using ansible
* First, enter this code block into a `VagrantFile`:
```ruby
# ansible-tech201


# -*- mode: ruby -*-
 # vi: set ft=ruby :
 
 # All Vagrant configuration is done below. The "2" in Vagrant.configure
 # configures the configuration version (we support older styles for
 # backwards compatibility). Please don't change it unless you know what
 
 # MULTI SERVER/VMs environment 
 #
 Vagrant.configure("2") do |config|
    # creating are Ansible controller
      config.vm.define "controller" do |controller|
        
       controller.vm.box = "bento/ubuntu-18.04"
       
       controller.vm.hostname = 'controller'
       
       controller.vm.network :private_network, ip: "192.168.33.12"
       
       # config.hostsupdater.aliases = ["development.controller"] 
       
      end 
    # creating first VM called web  
      config.vm.define "web" do |web|
        
        web.vm.box = "bento/ubuntu-18.04"
       # downloading ubuntu 18.04 image
    
        web.vm.hostname = 'web'
        # assigning host name to the VM
        
        web.vm.network :private_network, ip: "192.168.33.10"
        #   assigning private IP
        
        #config.hostsupdater.aliases = ["development.web"]
        # creating a link called development.web so we can access web page with this link instread of an IP   
            
      end
      
    # creating second VM called db
      config.vm.define "db" do |db|
        
        db.vm.box = "bento/ubuntu-18.04"
        
        db.vm.hostname = 'db'
        
        db.vm.network :private_network, ip: "192.168.33.11"
        
        #config.hostsupdater.aliases = ["development.db"]     
      end
    
    
    end
```
* Update and upgrade all 3 machines : controller, web, db
```
sudo apt update
```
```
sudo apt upgrade -y
```
* Need to install ansible on controller machine
1) Install python :
```
sudo apt-get install software-properties-common
```
2) Install ansible repository :
``` 
sudo apt-add-repository ppa:ansible/ansible
```
3) Run an update:
```
sudo apt update
```
4) Install ansible: 
```
sudo apt install ansible -y
```
5) Check the ansible version
```
ansible --v
```
6) SSH into web VE from controller, the numbers after @ are the ip for the web VE:
``` 
ssh vagrant@192.168.33.10
```
## Additional information
* Inside the controller VE where ansible is installed, the absolute oath is /etc/ansible
* This will have both the `hosts` and `ansible.conf` files

## Ansible
* Adhock commands can be used from the control manager (ansible terminal) to interact with agent nodes
* Can use ad-hock commands
* Scripts in ansible are called playbooks, and they are written in YAMAL language
* ` Vagrant status` -> outputs running state of every machine
* ` sudo apt update && upgrade -y` -> runs update and upgrade consequtively
* Can SSH into all 3 machines from local machine, that means communication is effective
* Now want to SSH into nodes from controller
* `
* Controller runs requist to web node
* Key wont match because key it isn't copied
* Need
1) On the controller VE, navigate to /etc/ansible
2) Edit the host file to allow the web machine:
```
192.168.33.10 ansible_connection=ssh ansible_ssh_user=vagrant ansible_ssh_pass=vagrant
```
3) Should now be able to run:
```
sudo ansible -m ping web
```
* This should return a positive `pong` response, which means that the two machines can communicate
## Potential blockers - Fail to use ssh key
* On the web VE:
1) Navigate to ssh folder:
```
cd /etc/ssh
```
2) Edit the sshd_config file:
```
sudo nano sshd_config
```
3) Make sure the following lines are uncommented, if they are not in the file add them:
* PermitRootLogin prohibit-password
* PasswordAuthentication yes
* ChallengeResponseAuthentication no
* UsePAM yes
* X11Forwarding yes
* PrintMotd no
* AcceptEnv LANG LC_*
* Subsystem       sftp    /usr/lib/openssh/sftp-server
* UseDNS no
* GSSAPIAuthentication no
4) Save and exit -> `CTRL + x` -> `y` -> `ENTER`
5) Restart ssh to save the changes:
```
sudo systemctl restart ssh
```
* On the controller VE:
1) Navigate to /etc/ansible
2) Edit the ansible.cfg file:
```
sudo nano ansible.cfg
```
3) Under [default], add the following line:
```
host_key_checking = false
```
4) Save and exit -> `CTRL + x` -> `y` -> `ENTER`
* You should now be able to ping the web node and receive a positive pong response
## modules
* Ping command tries to SSH in and checks the status code
* If status code is 200, gives response pong
* If the code is anything else, it gives the reason
* sudo ansible web -a "uname -a" -> runs uname -a in the target VE using ansible, in this case web
* sudo ansible web -a "date" -> prints the date of the target nodes location
* sudo ansible all -a "free -m" -> prints the availalble hardrive space in all VEs
*  sudo ansible all -a "ls -a" -> prints hidden files in target VE
* sudo ansible all -a "uptime"
* ansible web -m ansible.builtin.copy -a "src=test.txt dest=test.txt"

## AD hock benefit


## Provisioning with YAMAL

![](images/ansible_structure2.png)

### Requirements- db
1) Install mongodb
2) Configure mongodb
### Requirements- web
1) Install nginx 
2) Configure reverse proxy
3) Install nodejs
4) Install pm2
5) install npm
6) Make a persistent environmental variable with db ip
7) Seed the database