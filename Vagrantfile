# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"
  config.vm.hostname = "workshop-vm"

  config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "1024"
    vb.cpus = 2
  end

  config.vm.provision "shell", inline: <<-SHELL
    apt-get update
    apt-get upgrade -y
  SHELL

  config.vm.provision "docker", images: ["ubuntu:18.10"]

  config.vm.provision "shell", inline: <<-SHELL
    apt-get install -y docker-compose
  SHELL
end
