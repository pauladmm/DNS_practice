# -*- mode: ruby -*-
# vi: set ft=ruby :


Vagrant.configure("2") do |config|

  config.vm.box = "debian/bookworm64"

  config.vm.provision "shell", name: "update", inline: <<-SHELL
      apt-get update
      apt-get install -y bind9
    SHELL

  config.vm.define "venus" do |venus|
    venus.vm.box = "debian/bookworm64"
    venus.vm.hostname = "venus.sistema.test"
    venus.vm.network "private_network", ip: "192.168.57.102"

    
  end # venus

  config.vm.define "tierra" do |tierra|
    tierra.vm.box = "debian/bookworm64"
    tierra.vm.hostname = "tierra.sistema.test"
    tierra.vm.network "private_network", ip: "192.168.57.103"

    
  end # tierra
end
