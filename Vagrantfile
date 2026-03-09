# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "starboard/ubuntu-arm64-20.04.5"

  config.vm.provision "shell", path: "scripts/install.sh"

  config.vm.define "controlplane" do |cp|
    cp.vm.hostname = "controlplane"
    cp.vm.network "private_network", ip: "192.168.56.10"
    cp.vm.provider "vmware_desktop" do |v|
      v.vmx["memsize"]  = "2048"
      v.vmx["numvcpus"] = "2"
    end
  end

  config.vm.define "worker" do |w|
    w.vm.hostname = "worker"
    w.vm.network "private_network", ip: "192.168.56.11"
    w.vm.provider "vmware_desktop" do |v|
      v.vmx["memsize"]  = "2048"
      v.vmx["numvcpus"] = "2"
    end
  end
end
