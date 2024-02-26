# Arkime_ssl_ubuntu_deployment
Step-by-step guide for deploying Arkime with SSL on Ubuntu, ensuring secure network traffic monitoring and analysis. Includes detailed instructions and configuration setups.



## What is Arkime?
__Arkime__, formerly known as Moloch, is a large-scale, open-source, indexed packet capture and search tool designed to store and index network traffic in standard PCAP format. Arkime is accessed through a web interface or API and supports encrypting PCAP files at rest. 
More info on https://arkime.com/

__This is how the interface looks like:__
![Step27](https://github.com/samyank7/Arkime_ssl_ubuntu_deployment/assets/70804565/e5e49998-ba82-4d1c-849c-176b6ae05399)


## DEMO 
This is the official Demo from Arkime :
__The username and password are both__ - `arkime`
__Warning: Anyone can see anything you upload.__

```bash
https://demo.arkime.com/auth
```
## Requirements

Ubuntu 22.04 Desktop/server with 2GB RAM and 2 CPUs set up [minimum]

[Note that the amount of CPU, RAM, and storage that your Elasticsearch server will require depends on the volume of logs that you expect]

## Installation

### Step 1: Download the official Arkime file directly from its official website.             
Source: https://arkime.com/downloads

```bash
 wget https://s3.amazonaws.com/files.molo.ch/builds/ubuntu-22.04/arkime_5.0.0-1_amd64.deb
``` 
![Step1](https://github.com/samyank7/Arkime_ssl_ubuntu_deployment/assets/70804565/98779583-4516-452a-8ba2-4ee2f06d0022)

### Step 2: Installing Arkime

```bash
 sudo apt install ./arkime_5.0.0-1_amd64.deb 
```
![Step2(1)](https://github.com/samyank7/Arkime_ssl_ubuntu_deployment/assets/70804565/c3ea99f3-18dc-4f80-b19c-e968c6ee9fb0)

Visit the "opt" directory and verify whether the "arkime" folder exists there.

```bash
 cd /opt
```
```bash
 ls
```
![Step2(2)](https://github.com/samyank7/Arkime_ssl_ubuntu_deployment/assets/70804565/d6ce2482-3beb-4af7-ad2a-3a4dc83d2432)


### Step 3: Installing Elasticsearch

```bash
curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elastic.gpg
``` 
```bash
echo "deb [signed-by=/usr/share/keyrings/elastic.gpg] https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
``` 
```bash
sudo apt update
```
![Step3(1)](https://github.com/samyank7/Arkime_ssl_ubuntu_deployment/assets/70804565/f9d6fd4d-4c1f-40cb-b889-76160bb1bc19)

```bash
sudo apt install elasticsearch  
```
![Step3(2)](https://github.com/samyank7/Arkime_ssl_ubuntu_deployment/assets/70804565/23de8ce4-57ae-4fa6-8e46-435a09f2f337)


### Step 4: Configuring elasticsearch

Go to
```bash
sudo nano /etc/elasticsearch/elasticsearch.yml
```
In this file uncomment and edit the field 
```bash
network.host :<localhost or domain name>
```
![Step4(1)](https://github.com/samyank7/Arkime_ssl_ubuntu_deployment/assets/70804565/a0f97a41-1907-4db9-ab95-9598d3773133)

After making changes exit and  type the following commands to start elasticsearch and check the status
```bash
sudo systemctl start elasticsearch
``` 
```bash
sudo systemctl enable elasticsearch
``` 
```bash
sudo systemctl status elasticsearch
```
![Step4(2)](https://github.com/samyank7/Arkime_ssl_ubuntu_deployment/assets/70804565/53d02328-d1bc-459d-803d-cfb73a973e67)


### Step 5:  Updating firewall regulations and verifying the operational status of Elasticsearch.
```bash
sudo ufw allow 9200
```
```bash
sudo ufw start
```

```bash
sudo ufw reload
```

```bash
sudo ufw status
```
![Step5(1)](https://github.com/samyank7/Arkime_ssl_ubuntu_deployment/assets/70804565/b600473f-899b-4b08-987d-43b9f13367e6)

Testing Elastic search
```bash
curl -X GET 'http://localhost:9200'
```
![Step5(2)](https://github.com/samyank7/Arkime_ssl_ubuntu_deployment/assets/70804565/c059f3e8-04dd-4b83-bc87-1632d90a1df9)


### Step 6: SSL configuration for publicly available domain

__Skip this step if you are setting  up arkime on localhost__

```bash
sudo apt install certbot python3-certbot-apache 
```

```bash
certbot --version
```
Navigate to /etc/apache2/sites-available directory and run the following command to create a configuration file for your installation:

___Change the values for "your-domain" accordingly.___
```bash
sudo nano /etc/apache2/sites-available/your-domain.conf
```

Add the following content in the conf file:
```bash
<VirtualHost *:80>

ServerAdmin webmaster@your-domain.com

ServerName your-domain.com
ServerAlias www.your-domain.com
DocumentRoot /var/www/html/

<Directory /var/www/html/>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
</Directory>

ErrorLog ${APACHE_LOG_DIR}/your-domain.com_error.log
CustomLog ${APACHE_LOG_DIR}/your-domain.com_access.log combined

</VirtualHost>
```

Save the file and Exit.

Enable the Apache virtual host:
```bash
$ sudo a2ensite your-domain.conf
```
After that, restart the Apache web server.
```bash
$ sudo systemctl restart apache2
```
To get the SSL certificate using the Certbot, type the command given below:
```bash
sudo certbot --apache
```
You will be asked to provide your valid email address and accept the term of service:
```bash
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Enter email address (used for urgent renewal and security notices)
 (Enter 'c' to cancel): admin@your-domain.com
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.3-September-21-2022.pdf. You must
agree in order to register with the ACME server. Do you agree?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: Y
```
Next, you’ll be asked if you want to share your email with the Electronic Frontier Foundation to receive news and other information. If you do not want to subscribe to their content, write N.

```bash
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Would you be willing, once your first certificate is successfully issued, to
share your email address with the Electronic Frontier Foundation, a founding
partner of the Let's Encrypt project and the non-profit organization that
develops Certbot? We'd like to send you email about our work encrypting the web,
EFF news, campaigns, and ways to support digital freedom.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: N
```
Next, you will be asked to select the domain on which you want to install the Let’s Encrypt SSL
```bash
Account registered.

Which names would you like to activate HTTPS for?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: your-domain.com
2: www.your-domain.com
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate numbers separated by commas and/or spaces, or leave input
blank to select all options shown (Enter 'c' to cancel):
```
If the SSL certificate is successfully obtained, certbot displays a message to show the configuration was successful:
```bash
IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/your-domain.com.com/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/your-domain.com/privkey.pem
   Your cert will expire on 2023-03-22. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot
   again. To non-interactively renew *all* of your certificates, run
   "certbot renew"
 - Your account credentials have been saved in your Certbot
   configuration directory at /etc/letsencrypt. You should make a
   secure backup of this folder now. This configuration directory will
   also contain certificates and private keys obtained by Certbot so
   making regular backups of this folder is ideal.
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```
Now, you have successfully installed SSL on your website.
You can now open your website using https://, and you’ll notice a green lock icon.

Verifying Certbot Auto-Renewal
```bash
sudo systemctl status certbot.timer
```


### Step 7: SSL configuration for localhost

__Note: You can skip this step if you want work with arkime without SSL on localhost__

This guide will walk you through the steps to enable SSL on your local machine using self-signed certificates. This setup allows you to securely access your localhost server over HTTPS. 

Note- Due to SSL being locally installed it is likely that browser will show connection as insecure due to the use of a self-signed SSL certificate, which is considered non-trusted by default in modern browsers. 

Before getting started, make sure you have the following tools installed on your system:

- `libnss3-tools`: Command-line tools for Network Security Services (NSS)
- `mkcert`: A tool for creating locally-trusted development certificates

You can install these tools using the following commands:

Generate SSL Certificate:
```bash
sudo apt install libnss3-tools
```
```bash
sudo mkcert -install
```
```bash
sudo mkcert localhost
```
Move Certificate Files:
```bash
sudo mkdir -p /etc/ssl/certs /etc/ssl/private
```
```bash
sudo mv localhost.pem /etc/ssl/certs/
```
```bash
sudo mv localhost-key.pem /etc/ssl/private/
```
Enable Apache SSL Module:
```bash
sudo a2enmod ssl
```
```bash
sudo systemctl restart apache2
```
![Step6(2) png](https://github.com/samyank7/Arkime_ssl_ubuntu_deployment/assets/70804565/d9020f44-2f14-4eaa-ab5f-a2d7df7f6400)

Edit the default SSL configuration file:
```bash
sudo nano /etc/apache2/sites-available/default-ssl.conf
```
Add the following lines to specify the SSL certificate and key file paths:
```bash
SSLCertificateFile /etc/ssl/certs/localhost.pem
SSLCertificateKeyFile /etc/ssl/private/localhost-key.pem
```
![Step6(3) png](https://github.com/samyank7/Arkime_ssl_ubuntu_deployment/assets/70804565/8f7ebd82-954a-48e2-8077-dd65bba310fc)

Allow Traffic in Firewall:
```bash
sudo ufw allow in "Apache Full"
```
Enable Apache Headers Module and SSL:
```bash
sudo a2enmod headers
```
```bash
sudo a2ensite default-ssl
```
Restart Apache:
```bash
sudo systemctl restart apache2
```

#### Now your localhost server should be accessible over HTTPS. You can access it in your browser using https://localhost.
![Step6(4) png](https://github.com/samyank7/Arkime_ssl_ubuntu_deployment/assets/70804565/54906c57-cae4-4b0d-8575-ad5fb249e17c)


### Step 8: Configuring Arkime
Find your network interface
```bash
ip a
```
Start configuration script for Arkime 
```bash
/opt/arkime/bin/Configure
```
Select an interface to monitor;
```bash
Found interfaces: lo;enp0s3;enp0s8
Semicolon ';' seperated list of interfaces to monitor [eth1] enp0s8
```
The Configure script can install OpenSearch/Elasticsearch for you or you can install yourself select no because we have already installed ElasticSearch.

```bash
Install Elasticsearch server locally for demo, must have at least 3G of memory, NOT recommended for production use (yes or no) [no] no
```

Set Elasticsearch server URL, localhost:9200 in this setup. 
If not on localhost instead of Enter , type http://your-domain.com>:9200

```bash
Elasticsearch server URL [http://localhost:9200] ENTER

```
Set encryption password. Be sure to replace the password.
```bash
Password to encrypt S2S and other things [no-default] changeme
```

Change the HOST to your own domain name.

```bash
/opt/arkime/db/db.pl http://localhost:9200 init
```

Adding a admin user. [Set the required username, password accordingly]

```bash
/opt/arkime/bin/arkime_add_user.sh admin "Admin User" setaccordingly –admin
```

If you prefer creating  a normal user , refer to __/opt/arkime/bin/arkime_add_user.sh --help__ , for more info 


### Enable Arkime capture service

Note - This will start capture service on your host. 

Intialize Capture service
```bash
cd opt/arkime/bin
```
```bash
sudo ./arkime_update_geo.sh
```
```bash
sudo ./capture
```
Add the ufw rules:
```bash
sudo ufw allow 8005
```
```bash
sudo ufw reload
```

Starting the capture and viewer service:
```bash
systemctl start arkimecapture.service
```
```bash
systemctl start arkimeviewer.service
```

After this the website should be up and running at :
```bash
http:<domainname>:8005

For us it is : http://localhost:8005
```
Simply login with the credentials entered in this step.

__If you have skipped setting up SSL, your arkime instance is ready__

### Step 9: Activating SSL for Arkime  

__If you have activated your SSL on your publicliy available domain using certbot follow this:__ 

[replace the last name with your domain name entered while configuring ssl]

```bash
cd /etc/letsencrypt/live/<your domain name>
```

we need these two certificates: cert.pem and privkey.pem [Copy their path]

example : 
```bash
certFile= /etc/letsencrypt/live/<your domain>/cert.pem
keyFile= /etc/letsencrypt/live/<your domain>/privkey.pem 
```

Then go to
```bash
sudo nano /opt/arkime/etc/config.ini
```

and update the certfile and keyfile section with the path.
```bash
# How often to create a new OpenSearch/Elasticsearch index. hourly,hourly[23468],hourly12,daily,weekly,monthly

# See https://arkime.com/settings#rotateindex
rotateIndex=daily

# Uncomment if this node should process the cron queries and packet search jobs, only ONE node should
# process cron queries and packet search jobs
# cronQueries=true

# Cert file to use, comment out to use http instead
certFile= /etc/letsencrypt/live/<your domain>/cert.pem


# File with trusted roots/certs. WARNING! this replaces default roots
# Useful with self signed certs and can be set per node.
# caTrustFile=/opt/arkime/etc/roots.cert


# Private key file to use, comment out to use http instead

keyFile= /etc/letsencrypt/live/<your domain>/privkey.pem

```

__If you have activated your SSL for localhost using mkcert follow this:__ 


we need path for these two certificates: cert.pem and privkey.pem 

For our case our certificates are stores at:
```bash
certFile= /etc/ssl/certs/localhost.pem
keyFile= /etc/ssl/private/localhost-key.pem

```

Then go to
```bash
sudo nano /opt/arkime/etc/config.ini
```

and update the certfile and keyfile section with the path.
```bash
 How often to create a new OpenSearch/Elasticsearch index. hourly,hourly[23468],hourly12,daily,weekly,monthly

# See https://arkime.com/settings#rotateindex
rotateIndex=daily

# Uncomment if this node should process the cron queries and packet search jobs, only ONE node should
# process cron queries and packet search jobs
# cronQueries=true


# Cert file to use, comment out to use http instead
certFile= /etc/ssl/certs/localhost.pem


# File with trusted roots/certs. WARNING! this replaces default roots
# Useful with self signed certs and can be set per node.
# caTrustFile=/opt/arkime/etc/roots.cert


# Private key file to use, comment out to use http instead
keyFile= /etc/ssl/private/localhost-key.pem
```
![Step23](https://github.com/samyank7/Arkime_ssl_ubuntu_deployment/assets/70804565/7a6f3903-d47f-4b6b-a802-c45382a5b968)


### Step 10: Final Configurations and  check https for arkime

```bash
sudo systemctl daemon-reload
```
```bash
sudo systemctl restart arkimeviewer.service
```
```bash
sudo systemctl restart arkimeviewer.service
```

now go to and login with the credentials entered in step 8
```bash
https://<your-domain.com>:8005
```

For our case it is :
```bash
https://localhost:8005
```
![Step25](https://github.com/samyank7/Arkime_ssl_ubuntu_deployment/assets/70804565/015c385b-9393-41b4-9abb-055787390dc8)


### Step 11 :[Optional]Activating the upload button within the Arkime interface.

Go to :
```bash
sudo/opt/arkime/etc/config.ini
```
Uncomment the section that says UploadCommand and save the file.
```bash
# Specify the max number of indices we calculate spidata for.
# ES will blow up if we allow the spiData to search too many indices.
spiDataMaxIndices=4

# Uncomment the following to allow direct uploads. This is experimental
uploadCommand=/opt/arkime/bin/capture --copy -n {NODE} -r {TMPFILE} -c {CONFIG} {TAGS}
```
![Step24](https://github.com/samyank7/Arkime_ssl_ubuntu_deployment/assets/70804565/ba98aef3-650a-489d-a772-811f924a2900)


then restart capture and viewer services: 
```bash
sudo systemctl restart arkimecapture.service
```

```bash
sudo systemctl restart arkimeviewer.service
```
![Arkime](https://github.com/samyank7/Arkime_ssl_ubuntu_deployment/assets/70804565/cd9f65ba-b24f-4174-abc2-dc84d099cf33)






## References
Elasticsearch Config: 
https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-elasticsearch-on-ubuntu-22-04

Arkime Installation official guide:
https://raw.githubusercontent.com/arkime/arkime/main/release/README.txt


SSL for publicliy available domain: https://www.linuxtuto.com/how-to-install-apache-with-lets-encrypt-on-ubuntu-22-04/#

SSL for Localhost: https://www.arubacloud.com/tutorial/how-to-enable-https-protocol-with-apache-2-on-ubuntu-20-04.aspx

You can use this script to upload files via command line. It also clears all previous stored data. : 
https://gist.github.com/jstrosch/63910bdf7117f8f53a26227cfd56b6c6
