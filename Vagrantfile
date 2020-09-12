# -*- mode: ruby -*-
# vi: set ft=ruby :

servers = {
  'primary' => { 'ip' => '10.10.10.10', 'memory' => '2048', 'cpus' => '2', 'disk' => '/tmp/primary_disk.vdi' },
}

Vagrant.configure("2") do |config|

    servers.each do |name, conf|
      config.vm.define "#{name}" do |cfg|
        cfg.vm.box = "ewerton_silva00/centos-7-x86_64-minimal-2003"
        cfg.vm.box_version = "20200906"
        cfg.vm.box_check_update = true
        cfg.vm.hostname = "#{name}.meudominio.local"
        cfg.vm.network "private_network", ip: "#{conf['ip']}"
        cfg.vm.provision :hosts, :sync_hosts => true
  
        cfg.vbguest.auto_update = false
  
        cfg.vm.provider "virtualbox" do |vb|
          vb.name = "#{name}"
          vb.gui = false
          vb.cpus = "#{conf['cpus']}"
          vb.memory = "#{conf['memory']}"
          # --- Add second disk with size 10GB.
          vb.customize ['createhd', '--filename', "#{conf['disk']}", '--size', 10 * 1024 ]
          vb.customize ['storageattach', :id, '--storagectl', 'SATA', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', "#{conf['disk']}"]
        end
  
        cfg.vm.provision "shell", inline: <<-SHELL
          setenforce 0
          firewall-cmd --permanent --add-port=5432/tcp
          firewall-cmd --reload
          hostnamectl
        SHELL
      end
    end
end