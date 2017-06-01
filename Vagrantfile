# -*- mode: ruby -*-
# vi: set ft=ruby :

require "yaml"
settings = YAML::load_file "binaries.yml"
accounts = YAML::load_file "account.yml"

Vagrant.configure("2") do |config|
  config.vm.box = "bento/centos-6.7"
  config.vm.synced_folder ".", "/vagrant/"

  config.vm.provision "shell" do |s|
    s.inline = <<-SHELL
     echo "127.0.0.1 localhost" | tee /etc/hosts
     echo "192.168.100.10 automate" | tee -a /etc/hosts
     echo "192.168.100.20 chefserver" | tee -a /etc/hosts
     echo "192.168.100.30 buildnode" | tee -a /etc/hosts
     echo "192.168.100.40 compliance compliance" | tee -a /etc/hosts
    SHELL
  end

  config.vm.define "Chef Server" do |chef_server|
    chef_server.vm.hostname = "chefserver"
    chef_server.vm.network "private_network", ip: "192.168.100.20"
    chef_server.vm.provision "file", source: settings['chef-server']['package'], destination: settings["chef-server"]["tmp-path"]
    chef_server.vm.provision "file", source: settings['chef-manage']['package'], destination: settings["chef-manage"]["tmp-path"]
    chef_server.vm.provision "shell", inline: "rpm -Uvh #{settings['chef-server']['tmp-path']}"
    chef_server.vm.provision "shell", inline: "sudo chef-server-ctl reconfigure"
    chef_server.vm.provision "shell", inline: "sudo chef-server-ctl install chef-manage --path #{settings['chef-manage']['tmp-path']}"
    chef_server.vm.provision "shell", inline: "sudo chef-server-ctl reconfigure"
    chef_server.vm.provision "shell", inline: "sudo chef-manage-ctl reconfigure --accept-license"
    chef_server.vm.provision "shell", inline: "sudo chef-server-ctl user-create #{accounts['chef-server']['user-name']} #{accounts['chef-server']['first-name']} #{accounts['chef-server']['last-name']} #{accounts['chef-server']['email']} #{accounts['chef-server']['password']} --filename delivery.user"
    chef_server.vm.provision "shell", inline: "sudo scp delivery.user /vagrant/"
    chef_server.vm.provision "shell", inline: "sudo chef-server-ctl org-create deliveryorg 'my delivery organization' --filename ~/deliveryorg-validator.pem -a delivery"
    chef_server.vm.provider :virtualbox do |vb|
      vb.memory = 2048
      vb.cpus = 2
    end
  end

  config.vm.define "Build Node" do |build_node|
    build_node.vm.hostname = "buildnode"
    build_node.vm.network "private_network", ip: "192.168.100.30"
    build_node.vm.provider :virtualbox do |vb|
      vb.memory = 512
      vb.cpus = 1
    end
  end

  config.vm.define "Compliance Server" do |compliance|
    # Set hostname/fqdn otherwise it does not work
    compliance.vm.hostname = "compliance"
    compliance.vm.network "private_network", ip: "192.168.100.40"
    compliance.vm.provision "file", source: settings['compliance']['package'], destination: settings['compliance']['tmp-path']
    compliance.vm.provision "shell", inline: "rpm -Uvh #{settings['compliance']['tmp-path']}"
    compliance.vm.provision "shell", inline: "sudo chef-compliance-ctl reconfigure --accept-license"
    compliance.vm.provider :virtualbox do |vb|
      vb.memory = 1024
      vb.cpus = 2
    end
  end

  config.vm.define "Automate Server" do |automate|
    automate.vm.hostname = "automate"
    automate.vm.network "private_network", ip: "192.168.100.10"
    automate.vm.provision "file", source: settings["automate"]["package"], destination: settings["automate"]["tmp-path"]
    automate.vm.provision "file", source: "delivery.license", destination: "~/delivery.license"
    automate.vm.provision "shell", inline: "rpm -Uvh #{settings['automate']['tmp-path']}"
    automate.vm.provision "shell", inline: "sudo automate-ctl setup --license /home/vagrant/delivery.license --key /vagrant/delivery.user --server-url https://chefserver/organizations/deliveryorg --enterprise=#{accounts['automate']['enterprise-name']} --fqdn=automate --no-build-node --no-configure"
    automate.vm.provision "shell", inline: "sudo automate-ctl reconfigure"
    automate.vm.provision "shell", inline: "sudo automate-ctl create-enterprise #{accounts['automate']['enterprise-name']} --ssh-pub-key-file=/etc/delivery/builder_key.pub"
    # runner needs enterprise to install
    automate.vm.provision "shell", inline: "sudo automate-ctl install-runner buildnode vagrant -P vagrant -I /vagrant/#{settings['chefdk']['package']} -y -e #{accounts['automate']['enterprise-name']}"
    automate.vm.provision "shell", inline: "sudo automate-ctl reconfigure"
    automate.vm.provider :virtualbox do |vb|
      vb.memory = 4096
      vb.cpus = 4
    end

  end
end
