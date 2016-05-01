All the following should be done on your local workstation, 

Install delivery cli

Note, stop before the paragraph "Configure Delivery CLI", in the following page

https://docs.chef.io/ctl_delivery.html

Login to the delivery gui, and add a user
add an organisation eg "customer_test"

install chefdk, delivery cli, and git

type
```
delivery
git
chef --version
```
make sure all at latest versions, if not upgrade

Make a working directory.  This is where you will run the delivery project from.   You may have lots of different projects, so use one working directory per project.  For now, just use one directory. 

```
cd Customer/
mkdir Delivery
cd Delivery/
delivery --version
ls -al  ( `dir` for windows users )
mkdir .delivery
mkdir .chef
vi ./.delivery/cli.toml
```
add the following to the file `cli.toml` file
```
api_protocol = "https"
enterprise = "customer"
git_port = "8989"
organization = "myorg"
pipeline = "master"
server = "delivery.customer.chefdemo.net"
user = "delivery"
```
Login to chef server and obtain your private key and validator key
```
cp ~/Downloads/scott.pem ./.chef/
vi ./.chef/knife.rb
```
add the following content to your `knife.rb`
```
#See https://docs.getchef.com/config_rb_knife.html for more information on knife configuration options
current_dir = File.dirname(__FILE__)
log_level                :info
log_location             STDOUT
node_name                "scott"
client_key               "#{current_dir}/scott.pem"
validation_client_name   "customer-validator"
validation_key           "#{current_dir}/customer-validator.pem"
chef_server_url          "https://chef.customer.chefdemo.net/organizations/myorg"
cookbook_path            ["#{current_dir}/../cookbooks"]
knife[:supermarket_site] = 'https://supermarket.customer.chefdemo.net'
```
copy in your validator.pem
```
cp ~/downloads ./.chef/myorg-validator.pem
ignore mkdir cookbooks
ignore cd cookbooks/
ignore delivery token
chef gem list |grep knife-push
chef gem install knife-push --pre
```
this appears to not be working in windows chefdk. 
change the .gemrc file
```
:sources:
- http://jenkins.customer.co.uk/artifactory/api/gems/gems-remote/
:verbose: true
gem: --no-doc
```
ignore the warning about your path
```
knife node status
```
nothing happening with this?
```
knife status
knife ssl check
```
check if you trust the chef server certificate
if not, then retreive it.
```
knife ssl fetch
```
then check it is ok
```
knife status
knife node status
knife job status
```
Create a project in Delivery ( via the delivery gui )
Login to delivery ( as your username ), select an organistion ( demo ), and then create a new project ( chocngnaw )

if using bitbucket, refer to https://docs.chef.io/integrate_delivery_bitbucket.html

create a new repo in bitbucket, called "project_delivery"

note,  this section only required by first person to cerate the "project_delivery", you can try
```
delivery clone project_delivery
```

```
git clone https://bitbucket.my.url/scm/chef/project_delivery
cd project_delivery
git init
touch README.me
git add --all
git commit -m "Initial commit"
git push - u origin master
```
delivery config, for  bitbucket project in delivery gui
Project Key eg CHEF
repo name "project_delivery"

On your local workstation, do the following
```
mkdir chocngnaw
cd chocngnaw
delivery clone project_delivery
```
(the full command would be "delivery clone chocngnaw --ent=customer --org=demo --user=scott --server=delivery.customer.chefdemo.net" ), but most of this is already specified in our `./.delivery/cli.toml` file

download repo from bitbucket, need to fix the ssl cert issue.
```
git clone -c http.sslVerify=false clone https://stash.customer.co.uk/scm/chef/project_delivery
```
ssh to your delivery server, and accept the host hey ( ie the ssh host identifciation )
note, this has to be on port 8989, not the normal 22.  
```
ssh delivery.customer.chefdemo.net -p 8989
delivery token
```
Had error messages here, due to the private key not being in the correct place.  
we are on windows, and private keys were in C:\users\userid/.ssh    but the home drive is mapped to h:\, so we created .ssh on H:., and copied the files there.  We were running in a chefdk shell, powershell.   Error message below
```
Need to add the generic error message here
```

```
ls -al ./.delivery/
delivery setup
```
( The full command should be "delivery setup --ent=customer --org=demo --user=scott --server=delivery.customer.chefdemo.net", but again since this is specified in our `././delviery/cli.toml` it is not required )
```
cat ls -al ~/.delivery/api-tokens
echo "# chocngaw" >> README.md
git status
git add README
git status
git commit -m "Initial commit"
ls -al
```
Run delivery init, which will create an empty build cookbook for you (with an empty set of phase recipes), add the cookbook to your project, create the new pipeline and submit the project to Delivery for review: 
```
delivery init
```


