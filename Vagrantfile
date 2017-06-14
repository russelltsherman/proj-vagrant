# -*- mode: ruby -*-
# vi: set ft=ruby :

# Specify minimum Vagrant version and Vagrant API version
Vagrant.require_version '>= 1.9.5'
VAGRANT_API_VERSION = '2'
NET_INT = 'enp0s3'

# Require 'yaml' module
require 'yaml'

# Read YAML file with VM details (box, CPU, RAM, IP addresses)
# Edit VagrantMachines.yml to change VM configuration details
machines = YAML.load_file(File.join(File.dirname(__FILE__), 'VagrantMachines.yml'))

Vagrant.configure(VAGRANT_API_VERSION) do |config|

  # Use Vagrant's default insecure ssh key
  config.ssh.insert_key = false

  # Iterate through entries in YAML file to create VMs
  machines.each do |machine|
    config.vm.define machine['name'] do |srv|
      srv.vm.box = machine['box']
      srv.vm.hostname = machine['name']

      # Disable default synced folder
      srv.vm.synced_folder '.', '/vagrant', disabled: true

      # Configure the VM with RAM and CPUs per VagrantMachines.yml (VirtualBox)
      srv.vm.provider 'virtualbox' do |vb, override|
        vb.gui = false
        vb.memory = machine['ram']
        vb.cpus = machine['vcpu']
        vb.customize ["modifyvm", :id, "--groups", machine['vb_group']]
      end # srv.vm.provider virtualbox

      srv.vm.network "public_network",
        ip: machine['network']['public']['address'],
        netmask: machine['network']['public']['netmask']

      # default router
      srv.vm.provision "shell", inline: <<-SHELL
        sudo route add default gw #{machine['network']['public']['gateway']}
      SHELL

      # delete default gw on #{NET_INT}
      srv.vm.provision "shell", inline: <<-SHELL
        eval `route -n | awk '{ if ($8 =="#{NET_INT}" && $2 != "0.0.0.0") print "sudo route del default gw " $2; }'`
      SHELL

      # Enable provisioning with a shell script. Additional provisioners such as
      # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
      # documentation for more information about their specific syntax and use.
      srv.vm.provision "shell", inline: <<-SHELL
        ifconfig
        route
      SHELL

      # srv.vm.post_up_message = '
      # '

    end # config.vm.define
  end # machines.each
end