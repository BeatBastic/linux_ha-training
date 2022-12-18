# frozen_string_literal: true
require 'yaml'

# Required Vagrant plugins
plugins = [
  'hostmanager',
  'vbguest',
  'disksize',
]

# Nodes configuration is defined in config.yaml
config_file = File.join(__dir__, 'config.yaml')
# mount path for repo on VMs
code_dir = '/vagrant_linux_ha'
# Initial vms hash
vms = {}

# Sanity checks
unless File.exist?(config_file)
  puts 'File config.yaml not found. Must be in same dir of Vagrantfile'
  abort
end
plugins.each do |plugin|
  unless Vagrant.has_plugin?("vagrant-#{plugin}")
    puts "ERROR! Wir benoetigen das plugin vagrant-#{plugin}: vagrant plugin install vagrant-#{plugin}"
    abort
  end
end

# read config
config = YAML.load_file config_file

# parse config, merge defaults, add required data
config['nodes'].each do |node, conf|
  vms[node] = {}
  vms[node] = config['default'].merge conf
  vms[node]['fqdn'] = format('%<role>s.%<domain>s', role: node, domain: vms[node]['domain'])
  vms[node]['aliases'] = [
    format('%<role>s.%<domain>s %<role>s', role: node, domain: vms[node]['domain'])
  ]
end

# Vagrant configuration
Vagrant.configure('2') do |config|
  # defaults for all vms
  config.hostmanager.enabled = true
  config.hostmanager.manage_host = true
  config.hostmanager.ignore_private_ip = false
  config.hostmanager.include_offline = true
  config.vm.synced_folder '../', code_dir, mount_options: ['ro']
  # See https://github.com/mitchellh/vagrant/issues/1673
  config.ssh.shell = "bash -c 'BASH_ENV=/etc/profile exec bash'"

  # config per vm
  vms.each do |node, settings|
    config.vm.define settings['fqdn'] do |setting|
      setting.vm.box = settings['box']
      setting.vm.hostname = settings['fqdn']
      setting.vm.network :private_network, ip: settings['ip']
      setting.vbguest.installer_options = { allow_kernel_upgrade: true } if settings['box'].start_with?('centos', 'almalinux')
      setting.vm.provision 'shell', path: 'scripts/vagrant-sethostname.sh', args: settings['fqdn'].to_s
      setting.hostmanager.aliases = settings['aliases']
      setting.vm.provider 'virtualbox' do |v|
        file_to_disk = "disks/#{settings['fqdn']}-second_disk.vmdk"
        v.customize ['modifyvm', :id, '--name', settings['fqdn']]
        v.customize ['modifyvm', :id, '--cpus', settings['cpu'].to_s]
        v.customize ['modifyvm', :id, '--memory', settings['memory'].to_s]
        unless File.exist?(file_to_disk)
          v.customize [ "createmedium", "disk", "--filename", file_to_disk, "--format", "vmdk", "--size", 1024 * settings['additional_disk_size'] ]
        end
        v.customize [ "storageattach", settings['fqdn'] , "--storagectl", "SATA Controller", "--port", "2", "--device", "0", "--type", "hdd", "--medium", file_to_disk]
      end
    end
  end
end
