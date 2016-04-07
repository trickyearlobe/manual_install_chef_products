# Manually installing Delivery Build Nodes
This guide explains how to install Delivery build nodes by hand.

### Getting and installing the packages
Obtain and install the following packages using the instructions for [Getting and Installing Packages](./getting_packages.md)

* chefdk
* delivery-cli
* opscode-push-jobs-client

The following 3rd party packages are also required
* git (Should be in the OS's repo)
* runit (Can be obtained from https://packagecloud.io/imeyer/runit)

You will also need to install some Ruby gems which can be obtained from https://rubygems.org

    chef gem install knife-supermarket
    chef gem install knife-push

    # Optional for Sentry Raven
    # chef gem install sentry-raven

If you don't have internet access or a local gem server you will need to download them (and all their dependencies) manually and copy them over.

# Configuring the build node

### Bootstrap the buildnode
On your dev workstation which has been configured to talk to chef server.

    knife bootstrap <options> <nodename>

### Create a Push Jobs configuration
Create ```/etc/chef/push-jobs-client.rb``` with contents similar to the following

    LC_ALL='en_US.UTF-8'

    # Chef server connect options
    chef_server_url   'https://chef.myorg.chefdemo.net/organizations/myorg'
    node_name         'builder1.myorg.chefdemo.net'
    client_key        '/etc/chef/client.pem'
    trusted_certs_dir '/etc/chef/trusted_certs'
    verify_api_cert   true
    ssl_verify_mode   :verify_peer

    whitelist({
	     'chef-client'  => 'chef-client',
	     /^delivery-cmd (.+)$/=>"/var/opt/delivery/workspace/bin/delivery-cmd '\\1'"
    })

    # We're under runit, so don't output timestamp
    Mixlib::Log::Formatter.show_time = true

### Create a runit configuration to start Push Jobs daemon
Create the required directories

    mkdir -p /etc/sv/opscode-push-jobs-client/log/main
    mkdir -p /etc/sv/opscode-push-jobs-client/env
    mkdir -p /etc/sv/opscode-push-jobs-client/control
    mkdir -p /var/log/opscode-push-jobs-client

    touch /etc/sv/opscode-push-jobs-client/log/config
    ln -s /etc/sv/opscode-push-jobs-client/log/config /var/log/opscode-push-jobs-client/config

Create a script ```/etc/sv/opscode-push-jobs-client/run``` with 755 permissions

    #!/bin/sh
    exec 2>&1
    exec /opt/push-jobs-client/bin/pushy-client -l info  -c /etc/chef/push-jobs-client.rb

Create a script ```/etc/sv/opscode-push-jobs-client/log/run``` with 755 permissions

    #!/bin/sh
    exec svlogd -tt /var/log/opscode-push-jobs-client

Create a symlink in the init.d directory to sv

    ln -s /sbin/sv /etc/init.d/opscode-push-jobs-client

Create a symlink for pushy in the service directory

    ln -s /etc/sv/opscode-push-jobs-client /etc/service/opscode-push-jobs-client

### Create the dbuild user and workspace (home dir)

    mkdir -p /var/opt/delivery
    useradd dbuild -d /var/opt/delivery/workspace -m -s /bin/bash

    mkdir -p /var/opt/delivery/workspace/bin
    chown root:root /var/opt/delivery/workspace/bin

    mkdir -p /var/opt/delivery/workspace/etc
    chown root:root /var/opt/delivery/workspace/etc

    mkdir -p /var/opt/delivery/workspace/lib
    chown root:root /var/opt/delivery/workspace/lib

    mkdir -p /var/opt/delivery/workspace/.chef
    chown dbuild:dbuild /var/opt/delivery/workspace/.chef

    touch /var/opt/delivery/workspace/etc/delivery-git-ssh-known-hosts

### Lay down the builder credentials
Depending on version, two different locations are used for credentials. We will create both for safety.

Copy the delivery.pem obtained when creating the ```delivery``` user in Chef Server to the following locations:-
* ```/var/opt/delivery/workspace/etc/delivery.pem``` filemode 644 owner dbuild:root
* ```/var/opt/delivery/workspace/.chef/delivery.pem``` filemode 644 owner dbuild:root

Copy the builder SSH private key generated on the delivery server during creation of the enterprise to the following locations:-
* ```/var/opt/delivery/workspace/etc/builder_key``` filemode 600 owner dbuild:root
* ```/var/opt/delivery/workspace/.chef/builder_key``` filemode 600 owner dbuild:root

### Lay down the builder knife.rb/delivery.rb
The builder uses two locations (and names) for the knife.rb. Place the following contents in:-
* ```/var/opt/delivery/workspace/etc/delivery.rb``` filemode 600 owner dbuild:root
* ```/var/opt/delivery/workspace/.chef/knife.rb``` filemode 600 owner dbuild:root

The contents should be:-

    current_dir = File.dirname(__FILE__)
    eval(IO.read('/etc/chef/client.rb'))
    log_location STDOUT
    node_name "delivery"
    client_key "#{current_dir}/delivery.pem"
    trusted_certs_dir "/etc/chef/trusted_certs"

### Trust the Delivery SSL certificate
If Delivery is using self-signed SSL certificates you need to grab the SSL certificate and copy it to the build node under  ```/etc/chef/trusted_certs```

    openssl s_client -showcerts -connect delivery.myorg.chefdemo.net:443 </dev/null 2> /dev/null| openssl x509 -outform PEM > /etc/chef/trusted_certs/delivery.myorg.chefdemo.net

### Lay down delivery-cmd and git_ssh scripts
Copy [delivery-cmd](../reference/delivery-cmd) and [git_ssh](../reference/git_ssh) to ```/var/opt/delivery/workspace/lib``` and set permissions to 755

### Set /etc/chef permissions
The dbuild user needs access to some stuff in /etc/chef which is normally only available to root so:-

    chmod 755 /etc/chef
    chmod 644 /etc/chef/client.rb
    chmod 755 /etc/chef/trusted_certs
    chmod 644 /etc/chef/trusted_certs/*

### Mark the build node so delivery can find it
Delivery locates build nodes by searching the run list for the following:-
* delivery_build
* delivery_build::default
* delivery_builder
* delivery_builder::default

The official build node setup cookbook is ```delivery_build``` so to avoid clashes in future we will create a fake builder setup cookbook named ```delivery_builder``` and add it to the run list.

On your developer workstation navigate to your chef repo.

Create a blank cookbook and upload it to the chef server.

    chef generate cookbook delivery_builder
    knife cookbook upload delivery_builder

Add delivery_builder to the run list of the node

    knife node run_list set builder1.myorg.chefdemo.net "recipe[delivery_builder]"

Converge the build node so that the run list gets saved in the Chef server SOLR index.

    chef-client

NOTE: Failure to converge the build node will prevent the build node being found.
