# -*- mode: ruby -*-
# # vi: set ft=ruby :

# Specify minimum Vagrant version and Vagrant API version
Vagrant.require_version ">= 1.7.0"
VAGRANTFILE_API_VERSION = "2"

# Require YAML module
require 'yaml'
# Read YAML file with box details
NODES = YAML.load_file('nodes.yaml')

# Automatically manage plugins
REQUIRED_PLUGINS = %w( vagrant-auto_network vagrant-hosts vagrant-r10k )
REQUIRED_PLUGINS.each do |plugin|
    system "vagrant plugin install #{plugin}" unless Vagrant.has_plugin? plugin
end

DOMAIN = ".vagrant"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
    NODES.each do |name, data|
        config.vm.define name do |node|
            node.vm.provider :virtualbox do |vb|
                vb.cpus = data["cpus"]
                vb.memory = data["memory"]
                vb.name = name
            end
            node.vm.box = data["boxname"]
            node.vm.box_url = data["boxurl"]
            node.vm.network :private_network, :auto_network => true
            data["ports"].to_a.each do |port|
                node.vm.network :forwarded_port,
                host:  port["host"],
                guest: port["guest"]
            end
            node.vm.hostname = name + DOMAIN
            node.vm.provision :hosts
            node.r10k.puppet_dir = 'puppet'
            node.r10k.puppetfile_path = 'puppet/Puppetfile'
            node.r10k.module_path = 'puppet/environments/vagrant/modules_r10k'
            node.vm.provision :puppet do |puppet|
                puppet.environment_path = "puppet/environments"
                puppet.environment = "vagrant"
                puppet.hiera_config_path = "puppet/hiera.yaml"
                puppet.options = "--verbose --environment vagrant"
                puppet.working_directory = "/tmp/vagrant-puppet/environments"
                puppet.module_path = ["puppet/environments/vagrant/modules_r10k", "puppet/modules_dev"]
            end
        end
    end
end
