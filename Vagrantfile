# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "rockylinux/9"
  config.vm.box_version = "5.0.0"
  config.vm.hostname = "rocky-infra-01"

  # Configuración de red privada para el laboratorio DNS/DHCP
  config.vm.network "private_network", ip: "192.168.10.10"

  # Recursos recomendados para la máquina virtual
  config.vm.provider "virtualbox" do |vb|
    vb.name = "rocky-infra-01"
    vb.memory = "2048"
    vb.cpus = 2
  end
end
