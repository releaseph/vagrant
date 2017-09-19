# Vagrant Repository
This repository contains vagrant configurations for testing puppet configuration through virtual machines.
## Getting started
This repository requires that you have Vagrant installed and configured with you preferred virtual machine provider. See the [Getting Started](https://www.vagrantup.com/intro/getting-started/index.html) guide for installing and configuring Vagrant.
## The Vagrantfile
Once you have Vagrant properly setup, you may use the repository's Vagrantfile for launching virtual machines.

Each launched virtual machine has preset configurations provided by the Vagrantfile:
```ruby
$install_puppet = <<EOF
rpm -Uvh http://yum.puppetlabs.com/el/7/products/x86_64/puppetlabs-release-7-11.noarch.rpm
yum -y install puppet
EOF

Vagrant.configure("2") do |config|
  config.vm.box = "centos/7"

  config.vm.synced_folder "puppet-hiera/hieradata", "/etc/puppetlabs/code/environments/production/hieradata"
  config.vm.provision "shell", inline: $install_puppet

  [...]
end
```
1. Base image is 'centos/7'
2. The contents of the directory puppet-hiera residing in the same directory as the Vagrantfile is synced with the virtual machine on launch (more on this later)
3. Puppet (not puppetserver) is also installed at launch

## The Puppetmaster
As described above, puppet is installed and the contents of the puppet-hiera directory is synced with the virtual machine at launch time. This means that once the virtual machine boots up, it is prepared for getting configuration data from puppet-hiera.

This also means that the puppet-hiera directory is your puppetmaster for spinning up with Vagrant.

The puppet-hiera directory is actually a submodule pointing to [releaseph/puppet-hiera](https://github.com/releaseph/puppet-hiera), but is locked to a specific commit. If you made any changes in the puppet-hiera repository, make sure to update this repository's submodule. You may also commit the submodule update (you should see it as a change)

```
git submodule init
git submodule update
```

## Puppetizing virtual machines
The Vagrantfile also has further blocks that define the puppet configuration it will get.
```ruby
config.vm.define "tng-dev" do |instance|
  instance.vm.provision "puppet" do |puppet|
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
```
1. The 'config.vm.define "[name]"' defines the virtual machine name. The name is specified when executing 'vagrant up'
2. The block 'instance.vm.provision "puppet"' executes the [Puppet Apply Provisioner](https://www.vagrantup.com/docs/provisioning/puppet_apply.html). All configuration prefixed with 'puppet' points the virtual machine to use the puppet-hiera directory as its master
3. Under 'puppet.facter', the puppet_role and puppet_env are variables that are passed to and read by the puppetmaster (you will see the variables being declared in puppet-hiera/manifests/site.pp). As with the puppet control repository's design, these variables determnine the configuration that is passed to the virtual machine

## Launching and testing
You should be able to launch the instance using vagrant up
```
vagrant up [name]
```

When testing more configurations, you may simply copy an existing 'config.vm.define' block and change the following:
* Name of virtual machine (e.g. tng-dev as above)
* puppet_role (e.g. tng as above)
* puppet_env (e.g. dev as above)
