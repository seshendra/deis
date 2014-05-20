# -*- mode: ruby -*-
# # vi: set ft=ruby :

DEIS_NUM_INSTANCES = (ENV['DEIS_NUM_INSTANCES'].to_i > 0 && ENV['DEIS_NUM_INSTANCES'].to_i) || 1

if DEIS_NUM_INSTANCES == 1
  mem = 4096
  cpus = 2
else
  mem = 2048
  cpus = 1
end

Vagrant.configure("2") do |config|
  config.vm.box = "coreos-317.0.0"
  config.vm.box_url = "http://storage.core-os.net/coreos/amd64-usr/317.0.0/coreos_production_vagrant.box"

  config.vm.provider :vmware_fusion do |vb, override|
    override.vm.box_url = "http://storage.core-os.net/coreos/amd64-usr/317.0.0/coreos_production_vagrant_vmware_fusion.box"
  end

  config.vm.provider :virtualbox do |vb, override|
    # Fix docker not being able to resolve private registry in VirtualBox
    vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    vb.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
  end

  # plugin conflict
  if Vagrant.has_plugin?("vagrant-vbguest") then
    config.vbguest.auto_update = false
  end

  (1..DEIS_NUM_INSTANCES).each do |i|
    config.vm.define vm_name = "deis-#{i}" do |config|
      config.vm.hostname = vm_name

      config.vm.provider :virtualbox do |vb|
        vb.memory = mem
        vb.cpus = cpus
      end

      ip = "172.17.8.#{i+99}"
      config.vm.network :private_network, ip: ip

      # Enable NFS for sharing the host machine into the coreos-vagrant VM.
      config.vm.synced_folder ".", "/home/core/share", id: "core", :nfs => true, :mount_options => ['nolock,vers=3,udp']
      # FALLBACK use rsync if NFS has issues (mandatory if using Windows, or any Linux with an encrypted filesystem)
      # config.vm.synced_folder ".", "/home/core/share", type: "rsync"
      # Note that with rsync, local Deis code changes need to be re-synced to the VM by issuing a `vagrant rsync`

      # user-data bootstrapping
      config.vm.provision :file, :source => "contrib/coreos/user-data", :destination => "/tmp/vagrantfile-user-data"
      config.vm.provision :shell, :inline => "mv /tmp/vagrantfile-user-data /var/lib/coreos-vagrant/", :privileged => true
    end
  end

end
