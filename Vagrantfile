# -*- mode: ruby -*-
# vi: ft=ruby :

require 'rbconfig'
require 'yaml'

###############################################################################
#
# This is a generic Vagrantfile that can be used without modification in
# a variety of situations. Hosts and their properties are specified in
# `VagrantMachines.yml`.
#
###############################################################################

# Specify minimum Vagrant version and Vagrant API version
VAGRANT_MIN_VERSION = '>= 1.9.5'
VAGRANTFILE_API_VERSION = '2'
# Set your default base box here
DEFAULT_BASE_BOX = 'ubuntu/xenial64'
# Set Project name
PROJECT_NAME = '/' + File.basename(Dir.getwd)

###############################################################################
#
# This is a generic Vagrantfile that can be used without modification in
# a variety of situations. Hosts and their properties are specified in
# `VagrantMachines.yml`.
#
###############################################################################

machines = YAML.load_file(File.join(File.dirname(__FILE__), 'VagrantMachines.yml'))

def network_options(nw)
  options = {}

  if nw.key?('address')
    options[:ip] = nw['address']
    options[:netmask] = (nw['netmask'] ||= '255.255.255.0')
  else
    options[:type] = 'dhcp'
  end

  options[:mac] = nw['mac'].gsub(/[-:]/, '') if nw.key?('mac')
  options[:auto_config] = nw['auto_config'] if nw.key?('auto_config')
  options[:virtualbox__intnet] = true if nw.key?('intnet')
  options
end

Vagrant.require_version VAGRANT_MIN_VERSION
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  # Use Vagrant's default insecure ssh key
  config.ssh.insert_key = false

  # Iterate through entries in YAML file to create VMs
  machines.each do |host|
    config.vm.define host['hostname'] do |node|
      node.vm.box = host['box'] ||= DEFAULT_BASE_BOX
      node.vm.box_url = host['box_url'] if host.key? 'box_url'

      node.vm.hostname = host['hostname']
      node.vm.network 'public_network', network_options(host['network']['public'])
      node.vm.network 'private_network', network_options(host['network']['private'])

      # Disable default synced folder
      node.vm.synced_folder '.', '/vagrant', disabled: true

      if host.has_key?('synced_folders')
        host['synced_folders'].each do |folder|
          vm.synced_folder folder['src'], folder['dest'], folder['options']
        end
      end

      if host.has_key?('shell_always')
        host['shell_always'].each do |script|
          vm.provision "shell", inline: script['cmd'], run: "always"
        end
      end

      if host.has_key?('forwarded_ports')
        host['forwarded_ports'].each do |port|
          vm.network "forwarded_port", guest: port['guest'], host: port['host']
        end
      end

      node.vm.provider :virtualbox do |vb|
        vb.gui = false
        vb.cpus = (host['cpus'] || '1')
        vb.memory = (host['memory'] || '512')
        # WARNING: if the name of the current directory is the same as the
        # host name, this will fail.
        vb.customize ['modifyvm', :id, '--groups', PROJECT_NAME]
      end
    end
  end
end
