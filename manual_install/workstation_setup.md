```
cd Customer/
mkdir Delivery
cd Delivery/
delivery --version
ls -al
mkdir .delivery
mkdir .chef
vi ./.delivery/cli.toml
```
add the following to the file `cli.toml` file
```
api_protocol = "https"
enterprise = "customer"
git_port = "8989"
organization = "demo"
pipeline = "master"
server = "delivery.customer.chefdemo.net"
user = "scott"
```
# Login to chef server and obtain your private key and validator key
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
chef_server_url          "https://chef.customer.chefdemo.net/organizations/customer"
cookbook_path            ["#{current_dir}/../cookbooks"]
knife[:supermarket_site] = 'https://supermarket.customer.chefdemo.net'
```
copy in your validator.pem
```
cp ~/downloads ./.chef/westpacnz-validator.pem
ignore mkdir cookbooks
ignore cd cookbooks/
ignore delivery token
chef gem install knife-push --pre
knife node status
knife status
knife ssl check
knife ssl fetch
knife status
knife node status
knife job status
```
Create a project in Delivery ( via the delivery gui )
Login to delivery, select an organistion ( demo ), and then create a new project ( chocngnaw )

On your local workstation, do the following
```
mkdir chocngnaw
cd chocngnaw
delivery clone chocngnaw
```
(the full command would be "delivery clone chocngnaw --ent=customer --org=demo --user=scott --server=delivery.customer.chefdemo.net" ), but most of this is already specified in our `./.delivery/cli.toml` file
```
cd chocngnaw
delivery setup
ls -al ./.delivery/
delivery token
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
delviery init
```


