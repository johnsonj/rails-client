# -*- mode: ruby -*-
# vi: set ft=ruby :


#            _ _                _ _            _   
#  _ __ __ _(_) |___        ___| (_) ___ _ __ | |_ 
# | '__/ _` | | / __|_____ / __| | |/ _ \ '_ \| __|
# | | | (_| | | \__ \_____| (__| | |  __/ | | | |_ 
# |_|  \__,_|_|_|___/      \___|_|_|\___|_| |_|\__|
#   vagrant config for rails developers on windows
#
# To get started, run vagrant up --provider <hyperv|???>
#
## HOST PLATFORM NOTES ##
#
# Windows 8.1 (w/ Hyper-V) #################################
# Authentication:
#   During vagrant up, you're prompted for credentials for SMB (file sharing). These are the credentials to access your host machine.
#   If you're in a domain use the format user@domain for the username.
#   You can also specify this information via enviornment variables if you're comfortable with that.
# 
# Issues:
# => Support for configuring HyperV memory/CPU is pending and should be release with Vagrant 1.7.3 - https://github.com/mitchellh/vagrant/pull/5183
# => Local networking isn't working. The guest/dev env needs DHCP to spin up.
#
## ENV Variables ##
# mount_type          - 'smb' or 'nfs'
# smb_username (opt)  - local username for smb
# smb_password (opt)  - password for that user

# Stick private config stuff in this file if you'd like
require './private_files/vagrant_env.rb' if File.exists?('private_files/vagrant_env.rb')

Vagrant.require_version '>= 1.7.2'

VAGRANTFILE_API_VERSION = '2'

#
# Boiler plate vagrant setup 
#

needs_restart = false
plugins = {
  'vagrant-bindfs' => '0.3.2',
  'vagrant-hostmanager' => '1.5.0'
}
plugins.each do |plugin, version|
  # If Vagrant is being used in a bundle (local development)
  # We need to pull in the plugin directly
  if defined?(Bundler)
    require "bundler/shared_helpers"
    require plugin if Bundler::SharedHelpers.in_bundle? && !Vagrant.very_quiet?
  end

  unless Vagrant.has_plugin?(plugin)
    system("vagrant plugin install #{plugin} --plugin-version #{version}") || exit!
    needs_restart = true
  end

  exit system('vagrant', *ARGV) if needs_restart
end

def ansible_installed?
  require 'mkmf'
  find_executable 'ansible-playbook'
end

#
# Setup folder mount options
#

mount_opts = {type: ENV["mount_type"] || 'smb'}

if mount_opts[:type] == 'smb'
  mount_opts[:smb_username] = ENV["smb_username"]
  mount_opts[:smb_password] = ENV["smb_password"]
end

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  # ################################### #
  # Provider Specific [Resources]       #
  # ################################### #

  config.vm.provider :hyperv do |vb|
    # If you're not running 'edge' vagrant these params don't exist
    vb.memory     = 2 * 1024 if vb.respond_to? :memory=
    vb.maxmemory  = 4 * 1024 if vb.respond_to? :maxmemory=
    vb.cpus       = 2        if vb.respond_to? :cpus=
  end

  config.vm.provider :virtualbox do |vb|
    # Need to define memory/cpu here.
    # No virtual box specific config.. Add it!
  end

  # ################################### #
  # Base system                         #
  # ################################### #

  config.vm.box = 'hashicorp/precise64'
  config.ssh.forward_agent = true

  # ################################### #
  # Networking                          #
  # ################################### #

  config.vm.network 'private_network', type: 'dhcp'
  config.vm.define 'railsclient' do |machine|
    machine.vm.hostname = 'railsdev'
    machine.vm.network 'forwarded_port', guest: 80, host: 8080
    machine.vm.network 'forwarded_port', guest: 443, host: 8081
    machine.hostmanager.aliases = %w(railsclient.dev)
  end

  config.hostmanager.enabled = true
  config.hostmanager.manage_host = true
  config.hostmanager.ignore_private_ip = false
  config.hostmanager.include_offline = true

  # ################################### #
  # Synced folders                      #
  # ################################### #

  # Mount dev via rsync for performance. This is a one way sync so it clobbers the dev environment changes. 
  config.vm.synced_folder '../dev', '/railsclient/dev', type: 'rsync', rsync__auto: "true", rsync__exclude: ".git/", id: "dev"
  # Create a second copy of the dev instance for 'bi-directional' sync. This is for tasks that require output from the server, eg rails generators
  config.vm.synced_folder '../dev', '/railsclient/dev_presistent', mount_opts
  # Ansible needs access to the ansible config
  config.vm.synced_folder 'ansible', '/ansible', mount_opts

  # ################################### #
  # Provisioning                        #
  # ################################### #

  if ansible_installed?
    config.vm.provision 'ansible' do |ansible|
      ansible.playbook = 'ansible/site.yml'
      ansible.sudo = true
      ansible.groups = {
        'webservers'           => %w(railsclient),
        'dbservers'            => %w(railsclient),
        'development:children' => %w(webservers dbservers)
      }
      ansible.tags = ENV['TAGS']
    end
  else
    Dir['shell/*.sh'].each do |script|
      config.vm.provision 'shell', path: script, privileged: false
    end
  end
end