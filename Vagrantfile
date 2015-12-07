VAGRANTFILE_API_VERSION = "2"

require 'yaml'

################################################################################
# check plugins
################################################################################
required_plugins = %w(vagrant-hostmanager vagrant-hosts vagrant-vbguest)

plugins_to_install = required_plugins.select { |plugin| not Vagrant.has_plugin? plugin }
if not plugins_to_install.empty?
  puts "Installing plugins: #{plugins_to_install.join(' ')}"
  if system "vagrant plugin install #{plugins_to_install.join(' ')}"
    exec "vagrant #{ARGV.join(' ')}"
  else
    abort "Installation of one or more plugins has failed. Aborting."
  end
end

# Read YAML file with box details
servers = YAML.load_file('config/servers.yaml')

################################################################################
# define puppetmaster
################################################################################
puppetmaster = "puppet.arthurjames.vagrant"

################################################################################
# define puppet apply
################################################################################
default_env = 'production'
ext_env     = ENV['VAGRANT_PUPPET_ENV']
env         = ext_env ? ext_env : default_env
PUPPETAPPLY = "puppet apply --verbose --hiera_config /etc/puppet/hiera.yaml --modulepath=/etc/puppet/environments/#{env}/modules /vagrant/puppet/manifests/default.pp"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  ##############################################################################
  # Change hosts file, both on clients and host
  ##############################################################################
  config.hostmanager.enabled = true
  config.hostmanager.manage_host = true
  config.hostmanager.ignore_private_ip = false
  config.hostmanager.include_offline = true

  servers.each do |server|
    config.vm.provision :hosts do |provisioner|
      provisioner.add_host '10.0.11.2', ['puppet.arthurjames.vagrant']
    end

    config.vm.provision "puppet_server" do |puppet|
      puppet.puppet_server = "puppet.arthurjames.vagrant"
      puppet.puppet_node   = server["hostname"]
      puppet.options       = "--verbose --debug"
    end

    config.vm.define server["name"] do |srv|
      config.vm.hostname                = server["hostname"]
      srv.vm.box                        = server["box"]

      srv.vm.provider :virtualbox do |vb|
        vb.name = server["name"]
        vb.customize [
           'modifyvm', :id,
           '--groups', '/Monitoring',
           '--memory', server["ram"]
      ]
      end
  
      config.vm.network "private_network", ip: server["ip"]
      if server["ports"]
        server["ports"].each do |port|
          srv.vm.network "forwarded_port", host_ip: "127.0.0.1", guest: port["guest"], host: port["host"]
        end
      end
    end
  end
end
