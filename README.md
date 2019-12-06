# Useful Commands

## Useful Links
- [LZone.de](https://lzone.de/cheat-sheet/)
- [Kubernetes](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [Devdocs.io](https://devdocs.io)
- [DigitalOcean Working with SSL Certificates](https://www.digitalocean.com/community/tutorials/openssl-essentials-working-with-ssl-certificates-private-keys-and-csrs)
- [Keytool](https://www.sslshopper.com/article-most-common-java-keytool-keystore-commands.html)

## SSL Certificates 

#### Generate a private key and CSR
	openssl req -out CSR.csr -new -newkey rsa:2048 -nodes -keyout privateKey.key

#### Generate a self signed certificate
	openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -keyout privateKey.key -out certificate.crt

#### Generate a CSR for an existing certificate
	openssl req -out CSR.csr -key privateKey.key -new

#### Generate a CSR based on an existing certificate
	openssl x509 -x509toreq -in certificate.crt -out CSR.csr -signkey privateKey.key

#### Remove a passphrase from a private key.
	openssl rsa -in privateKey.pem -out newPrivateKey.pem

#### Check whether a CSR is valid
	openssl req -text -noout -verify -in CSR.csr

#### Check whether a private key is valid
	openssl rsa -in privateKey.key -check

#### Check whether a certificate is valid
	openssl x509 -in certificate.crt -text -noout

#### Check whether a PKCS12 file is valid (.pfx or .p12)
	openssl pkcs12 -info -in keyStore.p12

#### Check an MD5 hash of the public key to ensure that it matches with what is in a CSR or private key
	openssl x509 -noout -modulus -in certificate.crt | openssl md5
	openssl rsa -noout -modulus -in privateKey.key | openssl md5
	openssl req -noout -modulus -in CSR.csr | openssl md5

#### Check an SSL connection (displays all certificates including intermediate)
	openssl s_client -connect www.google.com:443

#### Check the expiry date of an SSL certificate.
	echo | openssl s_client -servername www.google.com -connect www.google.com:443 2>/dev/null | openssl x509 -noout -dates

#### Check SAN addresses used.
	echo | openssl s_client -connect www.google.com:443 | openssl x509 -noout -text | grep DNS:

#### Check certificate issuer used
	echo | openssl s_client -connect www.google.com:443 | openssl x509 -noout -text | grep Issuer

#### Convert DER file to a PEM
	openssl x509 -inform der -in certificate.cer -out certificate.pem

#### Convert a PEM file to a DER
	openssl x509 -outform der -in certificate.pem -out certificate.der

#### Convert a PKCS file containing a private key and certificates to a PEM
	openssl pkcs12 -in keyStore.pfx -out keyStore.pem -nodes

#### Convert a PEM file and a private key to a PKCS file
	openssl pkcs12 -export -out certificate.pfx -inkey privateKey.key -in certificate.crt -certfile CACert.crt

## Icinga

#### Commands
	service icinga status
	service icinga restart
	service icinga checkconfig
	service icinga show-errors

#### Paths
	cd /etc/nagios/
	cd /usr/lib64/nagios/plugins/

## Puppet

#### Puppet master
```
systemctl status puppetserver
puppet cert list
puppet cert list --all
puppet cert sign agent.domain.com
puppet --help | more
puppet help resource | more
pupper resource --types | more
puppet describe package | more
puppet config print | grep -i module
puppet module --help
puppet module generate author-modulename
puppet parser validate init.pp
```

#### Puppet agent
```
systemctl status puppet
systemctl start puppet
/etc/puppet/puppet.conf - Agent must have certname and server propeties
puppet agent -tv --noop
puppet agent -tv
```

#### Config file
	cat /etc/puppet/puppet.conf

#### Test only icinga
	puppet agent --noop -t --verbose --tags icinga
	puppet agent -t --noop --tags Icinga::Objects

#### Verify all tags
	/usr/bin/puppet agent -t --noop

#### Apply only icinga changes
	puppet agent --tags icinga


### MCollective
	mco puppet runonce --tags mcollective,icinga --noop -I '/HOTNAME_PREFIX([\d]+).HOTNAME_POSTFIX/' --batch 10 --batch-sleep 10
	mco puppet runonce --tags mcollective,icinga -I '/HOTNAME_PREFIX([\d]+).HOTNAME_POSTFIX/' --batch 10 --batch-sleep 10

### SSL issues on puppet managed machines
	rm -Rf /var/puppet/ssl

### CA Certificate Renewal

1. Backup original certs
```
cd /var/puppet/
cp -r ssl ssl.bk
```
2. Stop both Puppet & httpd
```
sudo /etc/init.d/httpd stop
```
3. Delete SSL folder
```
rm -rf /var/puppet/ssl
```
4. Disable auto sign
```
vi /etc/puppet/puppet.conf
```
5. Under master mark auto sign as false
```
autosign = false
```
6. Regenerate the CA and master's cert
```
sudo puppet cert list -a
sudo puppet master --no-daemonize --verbose
```
7. Start both Puppet & httpd
```
sudo /etc/init.d/httpd start
```
8. Backup & delete original DB certs(backup the SSL folder for each of the agents)
```
mv /etc/puppetdb/ssl /etc/puppetdb/ssl.bak
```
9. Generate new SSL 
```
/usr/sbin/puppetdb-ssl-setup -f
```
10. Restart PuppetDB
```
/etc/init.d/puppetdb restart
```
11. Remove all the certs from agents, script per environment
```
for i in $(cat server.txt | grep <ENV>); do ssh -o StrictHostKeyChecking=no -o ConnectTimeout=5 -q USER@$i "hostname -f && sudo rm -rf /var/puppet/ssl"; done >> log.<ENV>
```
12. Once complete, run puppet in noop to regenerate SSL 
```
for i in $(cat server.txt | grep <ENV>); do ssh -o StrictHostKeyChecking=no -o ConnectTimeout=5 -q USER@$i "hostname -f && sudo puppet agent -t --noop"; done >> log.<ENV>
```
13. Once all agents for all environments have generated new certs, sign on master
```
puppet cert sign --all
```
14. Re-run puppet on each agent to check if all servers signed correctly
```
for i in $(cat server.txt | grep <ENV>); do ssh -o StrictHostKeyChecking=no -o ConnectTimeout=5 -q USER@$i "hostname -f && sudo puppet agent -t --noop"; done >> log.<ENV>
```

## Java

### JConsle

#### Connect to remote tomcat through ssh
```
ssh -fN -D 7777 USER@hostname
jconsole -J-DsocksProxyHost=localhost -J-DsocksProxyPort=7777 service:jmx:rmi:///jndi/rmi://localhost:6969/jmxrmi
```

### Keytool

#### Commands for Creating and Importing

- Generate a Java keystore and key pair
	- ```keytool -genkey -alias mydomain -keyalg RSA -keystore keystore.jks -keysize 2048```

- Generate a certificate signing request (CSR) for an existing Java keystore
	- ```keytool -certreq -alias mydomain -keystore keystore.jks -file mydomain.csr```

- Import a root or intermediate CA certificate to an existing Java keystore
	- ```keytool -import -trustcacerts -alias root -file Thawte.crt -keystore keystore.jks```

- Import a signed primary certificate to an existing Java keystore
	- ```keytool -import -trustcacerts -alias mydomain -file mydomain.crt -keystore keystore.jks```

- Generate a keystore and self-signed certificate (see How to Create a Self Signed Certificate using Java Keytoolfor more info)
	- ```keytool -genkey -keyalg RSA -alias selfsigned -keystore keystore.jks -storepass password -validity 360 -keysize 2048```

#### Commands for Checking

- Check a stand-alone certificate
	- ```keytool -printcert -v -file mydomain.crt```

- Check which certificates are in a Java keystore
	- ```keytool -list -v -keystore keystore.jks```

- Check a particular keystore entry using an alias
	- ```keytool -list -v -keystore keystore.jks -alias mydomain```

#### Other Commands

- Delete a certificate from a Java Keytool keystore
	- ```keytool -delete -alias mydomain -keystore keystore.jks```

- Change a Java keystore password
	- ```keytool -storepasswd -new new_storepass -keystore keystore.jks```

- Export a certificate from a keystore
	- ```keytool -export -alias mydomain -file mydomain.crt -keystore keystore.jks```

- List Trusted CA Certs
	- ```keytool -list -v -keystore $JAVA_HOME/jre/lib/security/cacerts```

- Import New CA into Trusted Certs
	- ```keytool -import -trustcacerts -file /path/to/ca/ca.pem -alias CA_ALIAS -keystore $JAVA_HOME/jre/lib/security/cacerts```

## RabbitMQ

- Leaves erlang running but stops rabbitmq
	- ```rabbitmqctl stop_app```

- Stops rabbitmq and erlang
	- ```rabbitmqctl stop```

- Starts rabbitmq and erlang
	- ```rabbitmqctl start_app```

- Purge a specific queue on a specific cluster
	- ```rabbitmqctl purge_queue <queue name> -p <cluster name>```

- Join a specific cluster by connecting to one of the vhosts
	- ```rabbitmqctl join_cluster rabbit@<vhost on cluster>```

- Show the status of the cluster
	- ```rabbitmqctl cluster_status```

- Show the permissions on a specific cluster name
	- ```rabbitmqctl list_permissions -p <cluster name>```

- Rename the cluster
	- ```rabbitmqctl rename_cluster_node <old> <new>```

- Set the cluster name to be the domain name specific by facter (this is the default)
	- ```rabbitmqctl set_cluster_name "facter domain"```

## Yum/Rpm

- List software versions available via yum (i.e multiple versions of the same piece of software)
	- ```yum --showduplicates list <software name>```

- List all installed versions of a specific piece of software (identifies duplicated software)
	- ```rpm -qa | grep <software name>```

- Remove all cache data (this can be separated info: expire-cache, packages, headers, metadata, dbcache, rpmdb, plugins)
	- ```yum clean all```

- List software available for install along with the version and repo it would be installed from
	- ```yum list available```

- List all installed software along with the version and repo it was installed from
	- ```yum list installed```

- Search inside of rpm packages for a specific filename, identify the package containing said file
	- ```yum provides "*/<specific file>"```

- Download the file via yum, but do not install it
	- ```yum --downloadonly <software name>```

- Download the rpm package and all the additional packages required to install it
	- ```yumdownloader --resolve <software name>```

- Install a specific version of some software
	- ```yum install <software name>-1.2.3```

- Remove a specific version of some software (useful in the instance of duplicate software existing on a server)
	- ```yum remove <software name>-1.2.3```

- Give detailed information about a specific rpm
	- ```yum info <software name>```

- Search all the yum repositories for a specific piece of software
	- ```yum search <software name>```

- Check if there are any packages which can be updated via yum
	- ```yum check-update```

- List all the repos present on the server
	- ```yum repolist```

- Cat the content of each of the yum configuration files
	- ```cat /etc/yum.repos.d/*```

- Downgrade a specific piece of software which already exists on the server
	- ```yum downgrade <software name>-1.2.3```

- Reinstall a specific piece of software
	- ```yum reinstall <software name>```

- List rpms which have been grouped together
	- ```yum grouplist```

- List packages contained within the group
	- ```yum groupinfo <group name>```

- Install all the software specified in the group
	- ```yum groupinstall <group name>```

- Update everything excluding <software1> and <software2>
	- ```yum -x '<software1>' -x '<software2>' update```

- Fix the rpm -qa cache (this occurs when the cache contains incorrect information, often occurring after a repair)
	- ```rpm --rebuilddb```
	- ```rpmdb_verify Packages```

- Show the last installed programs and their date of installation
	- ```rpm -qa --last```

- List all files contained within an rpm
	- ```rpm -qlp <software name>.rpm```

- List all files contained with in installed rpm (list obtained via rpm -qa)
	- ```rpm -ql <installed package>```

- Show the rpm which was used to place /etc/shadow on the system
	- ```rpm -qf </etc/shadow>```

## SELinux
```
sestatus -v
sestatus -b | grep nagios
sestatus -b | grep nagios_unconfined_plugin_exec_t
getsebool -a
getsebool -a | grep nagios
ps -eZ | grep nagios_unconfined_plugin_t
semanage -l
semanage -l nagios_unconfined_plugin_t
semanage module -l nagios_unconfined_plugin_t
semanage module -l nagios_unconfined_plugin_exec_t
run sealert -l 8ce2ba39-e0d4-4f0e-9ed9-fa714703432f
sealert -l 8ce2ba39-e0d4-4f0e-9ed9-fa714703432f
ls -laZ /usr/lib64/nagios/plugins/arguments.rb
semanage fcontext -l nagios_unconfined_plugin_exec_t
semanage fcontext -l nagios_unconfined_plugin_exec_t | grep nagios
restorecon -RvF /etc/ssl/certs/
chcon unconfined_u:object_r:etc_t:s0 ./server.cer
```
