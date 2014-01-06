# -*- mode: ruby -*-
# vi: set ft=ruby :

require "vagrant"

if Vagrant::VERSION < "1.2.1"
  raise "This is only compatible with Vagrant 1.2.1+"
end

# Figure out project specific settings for items
project_config = File.exist?('.project.yml') ? YAML.load(File.read('.project.yml')) : {}
project_name = project_config['name'] || File.basename(Dir.getwd)
python_version = File.exist?('.python-version') ? File.read('.python-version') : '2.6.5'

chef_environment, chef_environments_path = if File.exist?('./.chef-environment')
  puts "Using local .chef-environment"
  [File.read('./.chef-environment').strip, '/tmp/environments']
elsif File.exist?('~/.chef-environment')
  puts "Using global .chef-environment"
  [File.read('~/.chef-environment').strip, '/tmp/environments']
end

# *** If you want to pull in newer upstream packages, dump them in ci-repo in 
# this repos folder and the package manager will be able to see them.
# NOTE You may still need to update dependencies to grab newer versions
ci_repo_dir = File.exist?("ci-repo") ? File.expand_path('ci-repo') : nil

Vagrant.configure("2") do |config|

  config.vm.hostname = "#{project_name}-build"

  # SendGrid Centos 6 box
  config.vm.define 'centos-6', primary: true do |c|
    c.berkshelf.berksfile_path = "./Berksfile"
    # TODO Should we randomize or increment IP address?  Does it make a diff 
    # to parallel runs?
    c.vm.network "private_network", ip: "192.168.100.2"
    c.vm.box = "sendgrid_centos-6.4_chef-11.8.0"
    c.vm.box_url = "http://repo.sendgrid.net/sendgrid_centos-6.4_chef-11.8.0.box"
  end

  config.vm.provider :virtualbox do |vb|
    # Give enough horsepower to build without taking all day.
    vb.customize [
      "modifyvm", :id,
      "--memory", "1536",
      "--cpus", "2"
    ]
  end

  # Ensure a recent version of the Chef Omnibus packages are installed
  # *** Enable if omnibus is used for packaging ***
#  config.omnibus.chef_version = :latest

  # Enable the berkshelf-vagrant plugin
  # *** Enable if using external cookbooks ***
  config.berkshelf.enabled = true
  # The path to the Berksfile to use with Vagrant Berkshelf
  config.berkshelf.berksfile_path = "./Berksfile"

  config.ssh.forward_agent = true

  host_project_path = Dir.getwd
  guest_project_path = "/home/vagrant/#{project_name}"

  config.vm.synced_folder host_project_path, guest_project_path, nfs: true

  if ci_repo_dir
    puts "Found ci-repo, linking for configuration"
    config.vm.synced_folder ci_repo_dir, "/tmp/ci-repo"
  end

  # prepare VM for build pipeline
  config.vm.provision :chef_solo do |chef|
    if chef_environment
      puts "Provisioning with environment: #{chef_environment} (path: #{chef_environments_path})"
      chef.environments_path = chef_environments_path
      chef.environment = chef_environment
    end

    chef.cookbooks_path = "cookbooks"
    chef.add_recipe "ci_dependencies"

    chef.json = {
      'python' => {
        'version' => python_version
      }
    }
  end
end
