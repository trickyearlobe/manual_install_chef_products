# Manually installing Chef Server
This guide explains how to install Chef server in a standard (non-HA, non-scaled) deployment with pushy, reporting and the enhanced management UI.

### Getting and installing the packages
Obtain and install the following packages using the instructions for [Getting and Installing Packages](./getting_packages.md)

* chef-server-core
* opscode-push-jobs-server
* opscode-reporting
* opscode-manage

### Configuring Chef server
All components have sensible defaults so it's often possible to start the server with a default configuration if you're not doing anything fancy.

* Base Chef server options live in [chef-server.rb](./reference/chef-server.rb.md)
* Push job server config lives in [opscode-push-jobs-server.rb](./reference/opscode-push-jobs-server.rb.md)
* Reporting configuration lives in [opscode-reporting.rb](./reference/opscode-reporting.rb.md)
* Management UI config lives in [chef-manage.rb](./reference/chef-manage.rb.md)

A set of common configuration scenarios are described below.

### Converging and starting the Chef server
Whenever the chef server config is changed you need to re-converge it to pick up the new config.

    chef-server-ctl reconfigure
    opscode-push-jobs-server-ctl reconfigure
    opscode-reporting-ctl reconfigure
    chef-manage-ctl reconfigure

You can check everything (re)started OK with the following commands

    chef-server-ctl status


# Common configuration scenarios

### Setting up Single Signon for Supermarket and Analytics with OAUTH2
OAUTH2 is an open standard for federation of authentication.

Chef server implements OAUTH2 so other Chef applications (such as Supermarket or Analytics) can authenticate users against the Chef user database using single signon (aka. SSO)

When the add-on application detects the user is not logged in it redirects the users browser to Chef server to sign in. Once the user successfully authenticates, Chef server redirects the users browser back to the add-on application.

If you want to add additional components like Supermarket or Analytics you need to ask Chef server to generate a shared uid/secret for each application and you need to supply a callback URI so Chef can redirect users back to the original application after authentication.

You can add the following lines to your ```/etc/opscode/chef-server.rb```

    # OAUTH2 configuration for Supermarket, Analytics and a third party app
    oc_id['applications'] = {
      'supermarket' => {
        'redirect_uri' => 'https://my.supermarket.server.fqdn/auth/chef_oauth2/callback'
      },
      'analytics' => {
        'redirect_uri' => 'https://my.analytics.server.fqdn/'
      },
      'my_cool_app' => {
        'redirect_uri' => https://cool.apps.org/my_cool_app/callback_oath2'
      }
    }

After converging the Chef Server, you should find three files in ```/etc/opscode/oc-id-applications``` called ```supermarket.json```, ```analytics.json``` and ```my_cool_app.json```.

They each contain a UID/SECRET that can be added to the configuration of the add-on application so it can participate in OAUTH2 with Chef server.

### Configuring Chef for LDAP authentication
Chef can authenticate users against an LDAP directory (including Windows Active Directory)

Chef needs to be able to access the LDAP directory and normally anonymous access to the directory is not allowed so a user will need to be created for Chef server. Once the user is created make the following configuration changes in ```/etc/opscode/chef-server.rb```

    # The root of the LDAP directory tree where we begin our search for users
    ldap['base_dn'] = 'OU=Employees,OU=Domain users,DC=example,DC=com'

    # The user ID and password that allow us to search the LDAP directory tree
    ldap['bind_dn'] = 'chef_server'
    ldap['bind_password'] = 'c0mpl3x_53crET'

    # You can restrict access based on group membership in AD or OpenLDAP with
    # the 'memberOf' overlay. The user must have the 'memberOf' attribute
    ldap['group_dn'] = 'CN=Chef Users,OU=Groups,DC=example,DC=com'
