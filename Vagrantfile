# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "rockylinux/9"
  config.vm.box_version = "5.0.0"

  # Servidor DNS y DHCP (rocky-infra-01)
  config.vm.define "server", primary: true do |server|
    server.vm.hostname = "rocky-infra-01"
    server.vm.network "private_network", ip: "192.168.10.10"

    server.vm.provider "virtualbox" do |vb|
      vb.name = "rocky-infra-01"
      vb.memory = "2048"
      vb.cpus = 2
    end
  end

  # Cliente de Red (rocky-client-01) - Recibe IP por DHCP desde rocky-infra-01
  config.vm.define "client" do |client|
    client.vm.hostname = "rocky-client-01"
    client.vm.network "private_network", type: "dhcp"

    client.vm.provider "virtualbox" do |vb|
      vb.name = "rocky-client-01"
      vb.memory = "1024"
      vb.cpus = 1
    end
  end
end
