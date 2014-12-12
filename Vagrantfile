# -*- mode: ruby -*-
# vi: set ft=ruby :
 
VAGRANTFILE_API_VERSION = "2"

VAGRANT_DEFAULT_PROVIDER = "virtualbox"

# freely inspired by
#   https://github.com/carmstrong/multinode-ceph-vagrant/blob/master/Vagrantfile
#   http://wiki.deimos.fr/Ceph_:_performance,_reliability_and_scalability_storage_solution
 
#Vagrant::Config.run do |config|
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # Default box OS
  config.vm.box = "ubuntu/trusty64"
  config.ssh.forward_agent = true
  unless ENV["VAGRANT_NO_PLUGINS"]
    required_plugins = %w( vagrant-hostmanager vagrant-cachier )
    required_plugins.each do |plugin|
      system "vagrant plugin install #{plugin}" unless Vagrant.has_plugin? plugin
    end

    # $ vagrant plugin install vagrant-hostmanager
    if Vagrant.has_plugin?("vagrant-hostmanager")
      config.hostmanager.enabled = true
    end
    # $ vagrant plugin install vagrant-cachier
    # Need nfs-kernel-server system package
    if Vagrant.has_plugin?("vagrant-cachier")
      config.cache.scope = :box
      config.cache.synced_folder_opts = {
        type: :nfs,
        # The nolock option can be useful for an NFSv3 client that wants to avoid the
        # NLM sideband protocol. Without this option, apt-get might hang if it tries
        # to lock files needed for /var/cache/* operations. All of this can be avoided
        # by using NFSv4 everywhere. Please note that the tcp option is not the default.
        mount_options: ['rw', 'vers=3', 'tcp', 'nolock']
      }
    end
  end

  # We need one Ceph admin machine to manage the cluster
  config.vm.define "ceph-admin" do |admin|
    admin.vm.hostname = "ceph-admin"
    admin.vm.network :private_network, ip: "172.21.12.10"
    admin.vm.provision :shell, :inline => "wget -q -O- 'https://ceph.com/git/?p=ceph.git;a=blob_plain;f=keys/release.asc' | apt-key add -", :privileged => true
    admin.vm.provision :shell, :inline => "echo deb http://eu.ceph.com/debian/ $(lsb_release -sc) main | tee /etc/apt/sources.list.d/ceph.list", :privileged => true
    admin.vm.provision :shell, :inline => "DEBIAN_FRONTEND=noninteractive apt-get update && apt-get install -yq ceph-deploy", :privileged => true
    admin.vm.provision :shell, :inline => "mkdir cluster-ceph", :privileged => false
  end

  # The Ceph client will be our client machine to mount volumes and interact with the cluster
  #config.vm.define "ceph-client" do |client|
  #  client.vm.hostname = "ceph-client"
  #  client.vm.network :private_network, ip: "172.21.12.11"
  #end

 # We provision 3 CEPH MON
  (1..3).each do |i|
    config.vm.define "ceph-mon-#{i}" do |config|
      config.vm.hostname = "ceph-mon-#{i}"
      config.vm.network :private_network, ip: "172.21.12.#{i+20}"
    end
  end

 # We provision 3 CEPH OSD
  (1..3).each do |i|
    config.vm.define "ceph-osd-#{i}" do |config|
      config.vm.hostname = "ceph-osd-#{i}"
      config.vm.network :private_network, ip: "172.21.12.#{i+30}"
      file_to_disk = 'osd-disk_' + "ceph-osd-#{i}" + '.vdi'
      config.vm.provider "virtualbox" do |vb|
        vb.customize ['createhd', '--filename', file_to_disk, '--size', 8 * 1024]
        vb.customize ['storageattach', :id, '--storagectl', 'SATAController', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', file_to_disk]
      end
    end
  end
end

