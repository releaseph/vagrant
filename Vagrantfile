$install_puppet = <<EOF
# rpm -Uvh https://yum.puppetlabs.com/puppetlabs-release-pc1-el-7.noarch.rpm
# Puppet 4 deprecates the --manifestdir option, which is what this vagrant configuration is using.
# Install Puppet 3.8 for testing
rpm -Uvh http://yum.puppetlabs.com/el/7/products/x86_64/puppetlabs-release-7-11.noarch.rpm
yum -y install puppet
EOF

Vagrant.configure("2") do |config|
  config.vm.box = "centos/7"

  # config.vm.provider "virtualbox" do |vb|
  #   vb.memory = "512"
  # end

  config.vm.synced_folder "puppet-hiera/hieradata", "/etc/puppetlabs/code/environments/production/hieradata"
  config.vm.provision "shell", inline: $install_puppet

  config.vm.define "tng-dev" do |instance|
    instance.vm.provision "puppet" do |puppet|
      # puppet.binary_path = '/opt/puppetlabs/bin'
      puppet.hiera_config_path = 'puppet-hiera/hiera.yaml'
      puppet.manifest_file = 'site.pp'
      puppet.manifests_path = 'puppet-hiera/manifests'
      puppet.module_path = 'puppet-hiera/modules'
      puppet.temp_dir = '/tmp/vagrant-puppet'
      puppet.facter = {
        'puppet_role' => 'tng',
        'puppet_env' => 'dev'
      }
    end
  end

  config.vm.define "cjo-testdeploy" do |instance|
    instance.vm.provision "puppet" do |puppet|
      # puppet.binary_path = '/opt/puppetlabs/bin'
      puppet.hiera_config_path = 'puppet-hiera/hiera.yaml'
      puppet.manifest_file = 'site.pp'
      puppet.manifests_path = 'puppet-hiera/manifests'
      puppet.module_path = 'puppet-hiera/modules'
      puppet.temp_dir = '/tmp/vagrant-puppet'
      puppet.facter = {
        'puppet_role' => 'cjo',
        'puppet_env' => 'testdeploy'
      }
    end
  end
end
