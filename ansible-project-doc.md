# Ansible-Project: Multi Tier Web Application Stack Setup Locally



![](/home/sagar/Pictures/Screenshot from 2024-03-08 11-50-38.png)



# **Prerequisite**

**PROJECT SETUP**

1. Oracle VM Virtualbox
2. Vagrant
3. Vagrant plugins
4. Ansible
5. Git bash or equivalent editor



# Step 1: Create a folder 

This is the environment where we will build our project

![](/home/sagar/Pictures/Screenshot from 2024-03-08 11-55-41.png)

# Step 2: VM Setup

- Install Vagrant using command

  ![](/home/sagar/Pictures/Screenshot from 2024-03-08 11-31-35.png)

- Navigate to the folder and install Ansible if not present run this command

  ![](/home/sagar/Pictures/Screenshot from 2024-03-08 12-00-24.png)

- Check the version of ansible and vagrant

  ![](/home/sagar/Pictures/Screenshot from 2024-03-08 12-01-37.png)

![](/home/sagar/Pictures/Screenshot from 2024-03-08 12-02-23.png)

- **Now create a Vagrantfile and insert these lines**

  #-*- mode: ruby -*-

  #vi: set ft=ruby :

  VAGRANTFILE_API_VERSION = "2"

  Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  #General Vagrant VM configuration

    config.vm.box = "generic/ubuntu1804"
    config.ssh.insert_key = false
    config.vm.synced_folder ".", "/vagrant", disabled: true
    config.vm.provider "virtualbox" do |vb|
      vb.memory = 512
      vb.linked_clone = true
    end

  #Database vm 

    config.vm.define "db01" do |db01|
      db01.vm.hostname = "db01.test"
      db01.vm.network  "private_network", ip: "192.168.56.2"

    end

  #Memcache vm

    config.vm.define "mc01" do |mc01|
      mc01.vm.hostname = "mc01"
      mc01.vm.network "private_network", ip: "192.168.56.3"
      mc01.vm.network "forwarded_port", guest: 11211, host: 11211
    end

  RabbitMQ vm

    config.vm.define "rmq01" do |rmq01|
      rmq01.vm.hostname = "rmq01"
      rmq01.vm.network "private_network", ip: "192.168.56.4"
    end

  #Tomcat vm

    config.vm.define "app01" do |app01|
      app01.vm.hostname = "app01.test"
      app01.vm.network  "private_network", ip: "192.168.56.5"
      app01.vm.provider "virtualbox" do |vb|
       vb.memory = "1024"
     end
   end

  #Nginx server

    config.vm.define "web01" do |web01|
      web01.vm.hostname = "db.test"
      web01.vm.network  "private_network", ip: "192.168.56.6"
    end
  end





- After saving this file run this command

  ![](/home/sagar/Pictures/Screenshot from 2024-03-08 12-10-46.png)

  

This script will create your virtual machines with allocated IP addresses and hostnames. It's a Ruby script commonly used for setting up Vagrant VMs, making it easier to monitor and troubleshoot your machines with just a few commands.

![](/home/sagar/Pictures/Screenshot from 2024-03-08 12-18-09.png)



- Now, we will configure our Ansible project to deploy a multi-tier architecture.

- First, we need to insert all our IP addresses into the host file located at /etc/ansible/hosts.

  ![](/home/sagar/Pictures/Screenshot from 2024-03-08 12-16-40.png)

- You can now ping and check if the VMs are reachable from your local machine.

  ![](/home/sagar/Pictures/Screenshot from 2024-03-08 12-21-57.png)

- We will be documenting all our architecture and setup in an Ansible playbook, which is the main focus of this project to understand Ansible in depth.

  ![](/home/sagar/Pictures/Screenshot from 2024-03-08 12-25-27.png)

This Ansible playbook snippet performs the following tasks:

1. It targets all hosts (`hosts: all`) and executes tasks with elevated privileges (`become: true`).

2. In the `pre_tasks` section, it defines a task named "Ensure system is updated".

3. The task uses the `apt` module to update the package cache (`update_cache: yes`). This ensures that the package manager has the latest information about available packages.

4. The task is conditional (`when:`) and will only be executed on hosts whose inventory hostname is not in the 'RabbitMq' group. This ensures that the update task is skipped for hosts designated for RabbitMQ.(RabbitMQ hosts may have specific update requirements or dependencies that differ from other hosts in the infrastructure. By excluding them from the general update task, you can tailor the update process to better suit their needs.)

   # Provisioning MySQL

   ![](/home/sagar/Pictures/Screenshot from 2024-03-08 12-31-11.png)

   - **name**: This is a descriptive name for the playbook task. It helps identify the task in the playbook.
   - **hosts**: Specifies the target hosts where the task will be executed. In this case, it's targeting hosts belonging to the "database" group.
   - **become**: Allows the task to run with escalated privileges. In this case, the task will run with root privileges (`sudo`).
   - **tasks**: This section contains a list of tasks to be executed on the target hosts.
     - **name**: A descriptive name for the task. It helps identify what the task does.
     - **apt**: Ansible module used to manage packages on Debian-based systems (such as Ubuntu). Here it's being used to install MySQL server.
       - **name**: Specifies the name of the package to install, in this case, `mysql-server`.
       - **state**: Specifies the desired state of the package. `present` indicates that the package should be installed if it's not already installed.

   This playbook snippet ensures that MySQL server is installed on hosts in the "database" group. It utilizes the `apt` module to manage packages on Debian-based systems and installs the `mysql-server` package.

   

![](/home/sagar/Pictures/Screenshot from 2024-03-08 12-32-21.png)

Here's a breakdown of each task:

- **Install pip3**:

  - **name**: Descriptive name for the task.

  - apt

    : Ansible module used to manage packages on Debian-based systems.

    - **name**: Specifies the name of the package to install, which is `python3-pip`.
    - **state**: Specifies the desired state of the package, which is `present` (install the package if it's not already installed).

- **Install Mysql module**:

  - **name**: Descriptive name for the task.

  - pip

    : Ansible module used to manage Python packages using pip.

    - **name**: Specifies the name of the Python package to install, which is `pymysql`.
    - **state**: Specifies the desired state of the package, which is `present` (install the package if it's not already installed).

- **Start and enable MySQL service**:

  - **name**: Descriptive name for the task.

  - service

    : Ansible module used to manage system services.

    - **name**: Specifies the name of the service to manage, which is `mysql`.
    - **state**: Specifies the desired state of the service, which is `started` (ensure the service is running).
    - **enabled**: Specifies whether the service should be enabled to start automatically on system boot, which is set to `yes`.