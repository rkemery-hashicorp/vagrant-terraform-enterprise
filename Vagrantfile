# -*- mode: ruby -*-
# vi: set ft=ruby :

# Check that the vagrant-disksize plugin is installed.
unless `vagrant plugin list`.split("\n").map { |x| x.split.first }.include?("vagrant-disksize")
    puts "Installing the vagrant-disksize plugin."
    `vagrant plugin install vagrant-disksize`
    puts "Please re-run `vagrant up` now!"
    exit 1
end

# Check that the JSON config files are valid by parsing them.
['settings.conf', 'replicated.conf'].each do |file|
  begin
    JSON.parse(File.read(file))
  rescue => JSON::ParserError
    puts "This configuration file contains invalid JSON: #{file}"
    exit 1
  rescue => Errno::ENOENT
    puts "The #{file} file does not exist, please provide a valid license and re-run."
    exit 1
  end
end

# Check that the license.rli file exists.
begin
  if File.read('license.rli').empty?
    puts "The license.rli file is empty, please provide a valid license and re-run."
    exit 1
  end
rescue => Errno::ENOENT
    puts "The license.rli file does not exist, please provide a valid license and re-run."
    exit 1
end

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
    config.vm.box = "ubuntu/bionic64"

    config.vm.box_check_update = true
  
    config.disksize.size = '50GB'
  
    config.vm.network "forwarded_port", guest: 8800, host: 8800, host_ip: "127.0.0.1"
    config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"
    config.vm.network "forwarded_port", guest: 443, host: 8443, host_ip: "127.0.0.1"
  
    config.vm.provider "virtualbox" do |vb|
      vb.gui = false
    
      vb.memory = "5120" # 1024 * 5 = 5GB
    end

    config.vm.provision "file", source: "./license.rli", destination: "/tmp/license.rli"
    config.vm.provision "file", source: "./settings.conf", destination: "/tmp/settings.conf"
    config.vm.provision "file", source: "./replicated.conf", destination: "/tmp/replicated.conf"

    config.vm.provision "shell", inline: <<-SHELL
      apt-get update -y 
      sudo mv /tmp/replicated.conf /etc/replicated.conf
      curl -o install.sh https://install.terraform.io/ptfe/stable
      sudo chmod +x install.sh
      sudo ./install.sh no-proxy no-public-address
    SHELL

    config.trigger.after :up do |trigger|
        if RUBY_PLATFORM.include? "darwin"
          trigger.info = "Opening Terraform Enterprise UI in Chrome"
          trigger.ruby do
            `if which open; then open -a "Google Chrome" https://localhost:8800; fi`
          end
        end
    end
end
