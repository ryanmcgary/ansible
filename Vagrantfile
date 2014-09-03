# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  config.vm.box = "precise64"
  config.vm.box_url = "http://files.vagrantup.com/precise64.box"
  config.vm.provision "shell", inline: "echo Hello"

  config.vm.define "golden1" do |conf|
    conf.vm.hostname = "golden1"
    conf.vm.box = "precise64"

    conf.vm.network :forwarded_port, guest: 80, host: 8081
    conf.vm.network :private_network, ip: "192.168.33.10"
    conf.vm.network :public_network
    conf.ssh.forward_agent = true
  end

  config.vm.define "golden2" do |conf|
    conf.vm.hostname = "golden2"
    conf.vm.box = "precise64"

    conf.vm.network :forwarded_port, guest: 80, host: 8082
    conf.vm.network :private_network, ip: "192.168.33.11"
    conf.vm.network :public_network
    conf.ssh.forward_agent = true
  end

  config.vm.define "golden3" do |conf|
    conf.vm.hostname = "golden3"
    conf.vm.box = "precise64"

    conf.vm.network :forwarded_port, guest: 80, host: 8083
    conf.vm.network :private_network, ip: "192.168.33.12"
    conf.vm.network :public_network
    conf.ssh.forward_agent = true
  end

end
