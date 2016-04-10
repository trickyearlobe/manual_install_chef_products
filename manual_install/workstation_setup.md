```
cd Westpac/
mkdir Delivery
cd Delivery/
delivery --version
ls -al
mkdir .delivery
mkdir .chef
vi ./.delivery/cli.toml
```
add the following to the file
```
api_protocol = "https"
enterprise = "westpacnz"
git_port = "8989"
organization = "demo"
pipeline = "master"
server = "delivery.westpacnz.chefdemo.net"
user = "scott"
```
# Login to chef server and obtain your private key and validator key
```
cp ~/Downloads/scott.pem ./.chef/
vi ./.chef/knife.rb
```
add the following content
```
#See https://docs.getchef.com/config_rb_knife.html for more information on knife configuration options
current_dir = File.dirname(__FILE__)
log_level                :info
log_location             STDOUT
node_name                "scott"
client_key               "#{current_dir}/scott.pem"
validation_client_name   "westpacnz-validator"
validation_key           "#{current_dir}/westpacnz-validator.pem"
chef_server_url          "https://chef.westpacnz.chefdemo.net/organizations/westpacnz"
cookbook_path            ["#{current_dir}/../cookbooks"]
knife[:supermarket_site] = 'https://supermarket.westpacnz.chefdemo.net'
```
copy in your validator.pem
```
cp ~/downloads ./.chef/westpacnz-validator.pem
mkdir cookbooks
cd cookbooks/
delivery token
chef gem install knife-push
knife node status
knife status
knife ssl check
knife ssl fetch
knife status
knife node status
knife job status
```
