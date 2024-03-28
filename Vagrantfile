# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # General Vagrant VM configuration
  config.vm.box = "generic/ubuntu1804"
  config.ssh.insert_key = false
  config.vm.synced_folder ".", "/vagrant", disabled: true
  config.vm.provider "virtualbox" do |vb|
    vb.memory = 512
    vb.linked_clone = true
  end

  # Database vm 
  config.vm.define "db01" do |db01|
    db01.vm.hostname = "db01.test"
    db01.vm.network  "private_network", ip: "192.168.56.2"

  end


### Memcache vm  #### 
  config.vm.define "mc01" do |mc01|
    mc01.vm.hostname = "mc01"
    mc01.vm.network "private_network", ip: "192.168.56.3"
    mc01.vm.network "forwarded_port", guest: 11211, host: 11211
  end


### RabbitMQ vm  ####
  config.vm.define "rmq01" do |rmq01|
    rmq01.vm.hostname = "rmq01"
    rmq01.vm.network "private_network", ip: "192.168.56.4"
  end


  # Tomcat vm
  config.vm.define "app01" do |app01|
    app01.vm.hostname = "app01.test"
    app01.vm.network  "private_network", ip: "192.168.56.5"
    app01.vm.provider "virtualbox" do |vb|
     vb.memory = "1024"
   end
 end

  # Nginx server
  config.vm.define "web01" do |web01|
    web01.vm.hostname = "db.test"
    web01.vm.network  "private_network", ip: "192.168.56.6"
  end
end

