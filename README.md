# Create-Website

## Setting up a website on a Debian machine from scratch using  
 * [HTTP](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol) and [optionally] [HTTPS](https://en.wikipedia.org/wiki/HTTPS)
 * [Nginx](https://nginx.org/en/docs/) as Web-Server
 * Namecheap for domain purchasing
 * [Letsencrypt](https://letsencrypt.org/) for SSL Certificate

## Objectives  
 * To set-up a website from scratch to without having to make any purchases [except for the server that will host your website], follow Steps 1 and 2. This will leave you with only an IP address and no FQDN [ie google.com, amazon.ca, etc] to access your website  
 * To set-up a website from scratch and also purchase and set up an FQDN, follow Steps 1, 2 and 3  
 * To set-up a website from scratch and also purchase and set up an FQDN as well generate the ssl certificate for allowing HTTPS requests, follow all the steps.  

[Basic Overall Steps to follow that I abstracted from to compose this README](http://www.howto-expert.com/how-to-get-https-setting-up-ssl-on-your-website/)

## Step 1: Install and set-up Nginx as the Web Server
Prereq questions you may have:  
 * [What is a Web-Server and its purpose?](https://en.wikipedia.org/wiki/Web_server) 
 * [What is ufw?](https://wiki.debian.org/Uncomplicated%20Firewall%20%28ufw%29)  

```shell

admin@ip-172-31-72-60:~$ sudo apt-get install -y nginx ufw # Installing Nginx and UFW


admin@ip-172-31-72-60:~$ sudo ufw allow "Nginx HTTP" # ensuring that the server will allow HTTP requests
Rules updated
Rules updated (v6)


admin@ip-172-31-72-60:~$ sudo ufw allow ssh # ensuring that the server will allow ssh connections
Rules updated
Rules updated (v6)

# Setting up UFW

admin@ip-172-31-72-60:~$ sudo systemctl enable ufw # Ensures that ufw will be started at the next boot automatically
Synchronizing state of ufw.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable ufw
admin@ip-172-31-72-60:~$ sudo systemctl start ufw
admin@ip-172-31-72-60:~$ sudo systemctl status ufw
● ufw.service - Uncomplicated firewall
   Loaded: loaded (/lib/systemd/system/ufw.service; enabled; vendor preset: enabled)
   Active: active (exited) since Mon 2018-03-05 08:44:24 UTC; 8s ago
     Docs: man:ufw(8)
  Process: 1764 ExecStart=/lib/ufw/ufw-init start quiet (code=exited, status=0/SUCCESS)
 Main PID: 1764 (code=exited, status=0/SUCCESS)

Mar 05 08:44:24 ip-172-31-72-60 systemd[1]: Starting Uncomplicated firewall...
Mar 05 08:44:24 ip-172-31-72-60 systemd[1]: Started Uncomplicated firewall.
admin@ip-172-31-72-60:~$ sudo ufw enable # it sets some internal state... it's tripped me before.
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup
admin@ip-172-31-72-60:~$ sudo ufw status # ensuring that the rules we wanted are implemented
Status: active

To                         Action      From
--                         ------      ----
Nginx HTTP                 ALLOW       Anywhere                  
22/tcp                     ALLOW       Anywhere                  
Nginx HTTP (v6)            ALLOW       Anywhere (v6)             
22/tcp (v6)                ALLOW       Anywhere (v6)             

admin@ip-172-31-72-60:~$ systemctl status nginx # determining status of nginx
● nginx.service - A high performance web server and a reverse proxy server
   Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
   Active: active (running) since Mon 2018-03-05 08:43:08 UTC; 2min 12s ago
     Docs: man:nginx(8)
 Main PID: 1650 (nginx)
    Tasks: 2 (limit: 4915)
   CGroup: /system.slice/nginx.service
           ├─1650 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
           └─1651 nginx: worker process

Mar 05 08:43:08 ip-172-31-72-60 systemd[1]: Starting A high performance web server and a reverse proxy server...
Mar 05 08:43:08 ip-172-31-72-60 systemd[1]: nginx.service: Failed to read PID from file /run/nginx.pid: Invalid argument
Mar 05 08:43:08 ip-172-31-72-60 systemd[1]: Started A high performance web server and a reverse proxy server.

Useful Commands
# starting nginx program if it is not already started
sudo systemctl start nginx

# restart the nginx program if it is already started and you need to make new changes take effect
sudo systemctl restart nginx
```

### Step 1.a: Configuring Amazon AWS Security Group
*Only Applicable if you are using AWS*  
  
Please note that AWS provides an extra layer of protection for any instances[aka VMs] that you spin up. To modify the protection to suit your needs, you will need to modify the *Security Group* that has been assigned to the instance you are setting up the website on.  
  
  1.a.I. [Go to AWS Console](https://console.aws.amazon.com/ec2)  

  1.a.II. Select the Instance you are setting the website on and click on the Security Group indicated in the box ![Selecting Instance](Picture%201.png)  

  1.a.III. Make sure that the rules listed under the "Inbound" Tabs include the ones in the picture here. If not, please add them exactly as shown. ![Inbound Rules](Picture%202.jpg)  

## Step 2: Configuring Nginx  
You will need to edit a file called the Nginx virtual hosts file. We will be editing this file in order to instruct nginx as to  
 * What is the folder that contains the index.html that we want to be displayed to the end-user
 * What is the port to listen on, port 80 is for HTTP and port 443 is for HTTPS. In this step, we will set up only port 80 for HTTPS

To determine other fun stuff you can do with the nginx virtual hosts file, feel free to refer to the [offical nginx documentation](https://nginx.org/en/docs/)    

Editing the Nginx virtual hosts file
```shell
sudo vim /etc/nginx/sites-available/default
```

if the above file does not exist, run the following command to determine where the file has been installed on your machine
```shell
sudo find / -name sites-available
```

Entries to change in the file are marked accordingly
```
server {
  listen 80 default_server;
  listen [::]:80 default_server;
  server_name _;

  root /path/to/folder/that/contains/the/index <--- NEEDS TO BE UPDATED

  index index.html index.htm index.nginx-debian.html; <-- the index file that is located in the folder stated above needs to be referenced here

  location / {
    try_files $uri $uri/ =404;
  }
}
}
```

Commands to allow the changes to take effect
```shell
sudo systemctl restart nginx #restarting nginx to allow changes to take effect
systemctl status nginx #checking status of nginx
```

You can now access the webpage at the Public IP/DNS of your machine which can be found on AWS like so:
![Public IP/DNS Name](Picture%203.png)  
  
## Step 3: Buying a domain
This section just requires you to buy a FQDN [Fully Qualified Domain Name] such as google.com, amazon.ca, reddit.com, etc.  
Two popular places to buy a domain name from are Namecheap and GoDaddy. More options are available [here](https://en.wikipedia.org/wiki/Category:Domain_registrars) and by googling "buy domain name"  
  
  Step 3.a: Looking for available domain
  
  Step 3.b: Purchase the domain and any add-ons you may want.  
  
  Step 3.c: Register the IP of your instance with the domain name you just purchased.  
*Please note that after this step, it may take a while for it to work as you will have to wait for [1] Namecheap to register the I.P. address that you gave it with the domain you purchased as well as [2] waiting for the DNS that your computer queries to determine the IP address associated with a domain to be updated to reflect the change. Step 2 may take a while [up to an hour] just based on how the DNS that your computer qwueries has been set-up to refresh its cache.*  
  
Here is the bare minimum you need to configure to be able to access your new website via NameCheap
![NameCheap DNS Configuration](Picture%204.jpg)


## Step 4: Obtaining an SSL Certificate using Letsencrypt
*You will need to know the webroot for your site, the webroot is the directory that houses all the files and folders that are serviced to the web server [which in our case, is Nginx]. If you are unsure, please refer to the file path indicate on the nginx config file we modified earlier on in Step 2 that starts with the word "root"*
Quick way to determine the root:

```shell
cat /etc/nginx/sites-available/default  | grep root
```


[Instructions on obtaining SSL Cert using LetsEncrypt are available here](https://certbot.eff.org/)

Known Issues and how to resolve them:
```shell
admin@ip-172-31-72-60:~$ sudo apt-get install python-certbot-nginx -t stretch-backports
Reading package lists... Done
E: The value 'stretch-backports' is invalid for APT::Default-Release as such a release is not available in the sources
```
Steps to Resolve [[extracted from this page](https://backports.debian.org/Instructions/)]
```shell
sudo bash -c 'echo "deb http://ftp.debian.org/debian stretch-backports main" >> /etc/apt/sources.list'
sudo apt-get update
```
  
Example of how to run the command:
```shell
aadmin@ip-172-31-72-60:~$ sudo certbot --authenticator webroot --installer nginx
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator webroot, Installer nginx
Enter email address (used for urgent renewal and security notices) (Enter 'c' to
cancel): ###########@gmail.com

-------------------------------------------------------------------------------
Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.2-November-15-2017.pdf. You must
agree in order to register with the ACME server at
https://acme-v01.api.letsencrypt.org/directory
-------------------------------------------------------------------------------
(A)gree/(C)ancel: A

-------------------------------------------------------------------------------
Would you be willing to share your email address with the Electronic Frontier
Foundation, a founding partner of the Let's Encrypt project and the non-profit
organization that develops Certbot? We'd like to send you email about EFF and
our work to encrypt the web, protect its users and defend digital rights.
-------------------------------------------------------------------------------
(Y)es/(N)o: N
No names were found in your configuration files. Please enter in your domain
name(s) (comma and/or space separated)  (Enter 'c' to cancel): jacemanshadi.ca
Obtaining a new certificate
Performing the following challenges:
http-01 challenge for jacemanshadi.ca
Input the webroot for jacemanshadi.ca: (Enter 'c' to cancel): /var/www/html
Waiting for verification...
Cleaning up challenges
Deployed Certificate to VirtualHost /etc/nginx/sites-enabled/default for jacemanshadi.ca

Please choose whether or not to redirect HTTP traffic to HTTPS, removing HTTP access.
-------------------------------------------------------------------------------
1: No redirect - Make no further changes to the webserver configuration.
2: Redirect - Make all requests redirect to secure HTTPS access. Choose this for
new sites, or if you're confident your site works on HTTPS. You can undo this
change by editing your web server's configuration.
-------------------------------------------------------------------------------
Select the appropriate number [1-2] then [enter] (press 'c' to cancel): 2
Redirecting all traffic on port 80 to ssl in /etc/nginx/sites-enabled/default

-------------------------------------------------------------------------------
Congratulations! You have successfully enabled https://jacemanshadi.ca

You should test your configuration at:
https://www.ssllabs.com/ssltest/analyze.html?d=jacemanshadi.ca
-------------------------------------------------------------------------------

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/jacemanshadi.ca/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/jacemanshadi.ca/privkey.pem
   Your cert will expire on 2018-06-03. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot again
   with the "certonly" option. To non-interactively renew *all* of
   your certificates, run "certbot renew"
 - Your account credentials have been saved in your Certbot
   configuration directory at /etc/letsencrypt. You should make a
   secure backup of this folder now. This configuration directory will
   also contain certificates and private keys obtained by Certbot so
   making regular backups of this folder is ideal.
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le

```

## Step 5: Changes to allow HTTPS requests

 Step 5.a: Configuring ufw to allow HTTPS requests  
  
```shell
admin@ip-172-31-72-60:~$ sudo ufw allow "Nginx HTTPS"
Rule added
Rule added (v6)
admin@ip-172-31-72-60:~$ sudo ufw status
Status: active

To                         Action      From
--                         ------      ----
Nginx HTTP                 ALLOW       Anywhere                  
22/tcp                     ALLOW       Anywhere                  
Nginx HTTPS                ALLOW       Anywhere                  
Nginx HTTP (v6)            ALLOW       Anywhere (v6)             
22/tcp (v6)                ALLOW       Anywhere (v6)             
Nginx HTTPS (v6)           ALLOW       Anywhere (v6)             

```

 Step 5.b: Configure the AWS Security Group to allow HTTPS requests
![AWS Security Group Configuration](Picture%205.jpg)

## Step 6: Makinng changes to your site  
  
You can now start actually working on the contents of your website!  
You can either modify the provided index that nginx set up for you located at the root folderyou determined in [Step 4](https://github.com/modernNeo/create-website#step-4-obtaining-an-ssl-certificate-using-letsencrypt), or you can place your root folder elsewhere and redirect nginx there by changing the folder assigned to "root" in /etc/nginx/sites-available/default and then restarting nginx.

Commands to allow the changes to take effect
```shell
sudo systemctl restart nginx #restarting nginx to allow changes to take effect
systemctl status nginx #checking status of nginx
```


If you are curious, you can take a look at the changes that `certbot` automatically did to your Nginx configuration file to [1] Implement the HTTP requests and [2] redirect HTTP request to HTTPS
Below are some References Pages for HTTPS Configuration that the `cerbot` command automatically performed  
[Reference Page for Setting up HTTPS](https://www.digicert.com/csr-ssl-installation/nginx-openssl.htm)  
[Reference Page for redirecting HTTP to HTTPS](https://www.digitalocean.com/community/questions/best-way-to-configure-nginx-ssl-force-http-to-redirect-to-https-force-www-to-non-www-on-serverpilot-free-plan-by-using-nginx-configuration-file-only)  

[Nginx Reference Page](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-16-04)  
UFW Reference Pages  
 * [Ubuntu](https://help.ubuntu.com/community/UFW)  
 * [Debian](https://wiki.debian.org/Uncomplicated%20Firewall%20%28ufw%29)