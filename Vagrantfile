# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.

# Vagrantfile API/syntax version.
VAGRANTFILE_API_VERSION = "2"
VM_IP="10.11.12.30"
VM_NAME="nhm-windshaft"
VM_MEMORY_LIMIT=4096
VM_CPUS=2
VM_CPU_LIMIT=100

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  config.vm.box = "ubuntu/precise64"
  config.vm.hostname = VM_NAME
  config.vm.network :private_network, ip: VM_IP

  config.vm.provider :virtualbox do |v|
    v.customize ["modifyvm", :id, "--memory", VM_MEMORY_LIMIT]
    v.customize ["modifyvm", :id, "--cpus", VM_CPUS]
    v.customize ["modifyvm", :id, "--cpuexecutioncap", VM_CPU_LIMIT]
  end

  # Mount directory at /var/www
  config.vm.synced_folder ".", "/var/www/nhm-windshaft", :mount_options => ["dmode=777,fmode=777"]

  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "ansible/playbook.yml"
    ansible.inventory_path = "ansible/inventories/development.ini"
    ansible.limit = 'all'
  end     
  
end
