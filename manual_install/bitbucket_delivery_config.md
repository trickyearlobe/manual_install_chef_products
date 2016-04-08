Based on this..

https://docs.chef.io/integrate_delivery_bitbucket.html

```
update-ca-trust force-enable
cd /etc/pki/ca-trust/source/anchors/
openssl s_client -showcerts -connect {BITBUCKET_SERVER}:443 </dev/null 2>/dev/null|openssl x509 -outform PEM >{BITBUCKET_SERVER}.crt
update-ca-trust extract
```
add customer ca, as unable to add bitbucket server through the gui
```
/etc/pki/ca-trust/source/anchors/mycacert.crt
update-ca-trust extract
```
Add the following to login to bitbucket
URL:  https://cvs.customer.co.uk
User: service-account
Pass: mysupersecretpassword

