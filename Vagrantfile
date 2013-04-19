require 'berkshelf/vagrant'
require 'yaml'

settings = YAML::load_file(File.expand_path("../settings.yml", __FILE__))
amis = { # http://cloud-images.ubuntu.com/locator/ec2/
  "ap-northeast-1" => "ami-fe6ceeff",
  "ap-southeast-1" => "ami-64084736",
  "ap-southeast-2" => "ami-04ea7a3e",
  "eu-west-1"      => "ami-ce7b6fba",
  "sa-east-1"      => "ami-a3da00be",
  "us-east-1"      => "ami-d0f89fb9",
  "us-west-1"      => "ami-fe002cbb",
  "us-west-2"      => "ami-70f96e40",
}

Vagrant.configure("2") do |config|
  config.berkshelf.enabled = true
  config.vm.box = "precise64"
  config.vm.box_url = "http://files.vagrantup.com/precise64.box"

  config.vm.provider :virtualbox do |vb, override|
    vb.customize ["modifyvm", :id, "--memory", "512"]
    vb.customize ["modifyvm", :id, "--cpus", 4]
    override.vm.hostname = "zabbix"
    override.vm.network :private_network, ip: "192.168.39.10"
  end

  config.vm.provider :aws do |aws, override|
    aws.access_key_id             = settings['aws']['access_key_id']
    aws.secret_access_key         = settings['aws']['secret_access_key']
    aws.keypair_name              = settings['aws']['keypare_name']
    override.ssh.private_key_path = settings['aws']['private_key_path']
    aws.security_groups           = settings['aws']['security_group']
    aws.instance_type             = settings['aws']['instance_type']
    aws.availability_zone         = settings['aws']['availability_zone']
    aws.ami                       = amis[settings['aws']['availability_zone']]
    override.ssh.username         = "ubuntu"
  end

  config.vm.provision :shell, :inline => "apt-get update"
  config.vm.provision :shell, :inline => "apt-get install -y ruby1.9.3 build-essential"
  config.vm.provision :shell, :inline => "gem install --no-rdoc --no-ri chef -v 10.24.0"

  config.vm.provision :chef_solo do |chef|
    chef.add_recipe "apt"
    chef.add_recipe "mysql::server"
    chef.add_recipe "zabbix"
    chef.add_recipe "zabbix::server"

    chef.json = {
      :mysql => settings['mysql'],
      :zabbix => {
        :agent => {
          :servers => ['localhost'],
        },
        :server => {
          :install => true,
          :dbpassword => settings["zabbix"]["server"]["dbpassword"],
        },
        :web => {
          :install => true,
          :fqdn => settings["zabbix"]["web"]["fqdn"],
        },
      },
    }
  end
end
