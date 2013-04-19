ec2-zabbix
==========

## Install

1. Install Vagrant 1.1+
2. cp and edit settings.yml

Then, run following commands:

    $ vagrant plugin install vagrant-aws
    $ vagrant plugin install berkshelf-vagrant
    $ vagrant up --provider=aws
