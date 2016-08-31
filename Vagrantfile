# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/xenial64"
  config.vm.box_check_update = false
  config.vm.hostname = "machine-learning"

  config.vm.provider "virtualbox" do |vb|
    vb.name = "machine-learning"
    vb.memory = 2048
    vb.cpus = 2
    #vb.gui = true
  end

  config.vm.provision "shell", inline: <<-SHELL
    apt-add-repository ppa:ansible/ansible
    apt-get update
    apt-get install -y ansible vim git
  SHELL
end
