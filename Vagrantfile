# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"
LOCAL_HTTP_PROXY = 'http://proxy:8080'

$script = <<SCRIPT
    echo Running Ansible provisioning
    cd playbook-artifactory
    ansible-galaxy install -r requirements.yml --force
    ansible-playbook site.yml -c local
SCRIPT
#require './vagrant-provision-reboot-plugin'

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  if Vagrant.has_plugin?('vagrant-proxyconf')
    config.apt_proxy.http     = LOCAL_HTTP_PROXY
    config.apt_proxy.https    = LOCAL_HTTP_PROXY
    config.proxy.no_proxy = 'localhost,127.0.0.1,192.168.56.*'
  end

  config.vm.define :packer  do |packer|

    packer.vm.box = "ubuntu/trusty64"
    packer.vm.network "private_network",
      ip: "192.168.56.4"
    packer.vm.network "forwarded_port", guest: 80, host: 80
    packer.vm.network "forwarded_port", guest: 8081, host: 8081
    packer.vm.hostname = "packer"

    packer.vm.box_check_update = false

    packer.vm.provider "virtualbox" do |vb|
      #vb.gui = true
      vb.name = "packer"
      vb.cpus = 1
      vb.memory = 2048
    end

    packer.ssh.forward_agent = true
    packer.ssh.shell = "bash -c 'BASH_ENV=/etc/profile exec bash'"

    packer.vm.provision :shell, path: "provision/install-ansible.sh"

    packer.vm.synced_folder "./", "/vagrant", disabled: true
    packer.vm.synced_folder "playbooks/", "/home/vagrant/playbooks", mount_options: ["dmode=777","fmode=666"]

    packer.vm.provision :shell, inline: "ansible-playbook playbooks/docker.yml -c local"
    packer.vm.provision :shell, inline: $script

  end

end
