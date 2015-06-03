# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  # Every Vagrant virtual environment requires a box to build off of.
  config.vm.box = "hashicorp/precise32"
  config.vm.hostname = "jekyll"
  config.vm.define "github-pages" do |base|
  end

  # Throw in our provisioning script
  config.vm.provision "shell", path: "bootstrap.sh", privileged: false

  # Map localhost:4000 to port 4000 inside the VM
  config.vm.network "forwarded_port", guest: 4000, host: 4000
  config.vm.network "private_network", ip: "192.168.50.60"

  # VirtualBox-specific configuration
  config.vm.provider :virtualbox do |vb|
    vb.customize ['modifyvm', :id, '--memory', 512, '--ioapic', 'on', '--cpus', 1, "--natdnshostresolver1", "on", "--natdnsproxy1", "on"]
    vb.customize ['guestproperty', 'set', :id, '/VirtualBox/GuestAdd/VBoxService/--timesync-interval',      1000]
    vb.customize ['guestproperty', 'set', :id, '/VirtualBox/GuestAdd/VBoxService/--timesync-min-adjust',    500]
    vb.customize ['guestproperty', 'set', :id, '/VirtualBox/GuestAdd/VBoxService/--timesync-set-threshold', 60000]
  end

  # Create a shared folder between guest and host
  config.vm.synced_folder "www/", "/srv/www", create: true

  config.ssh.forward_agent = true
end
