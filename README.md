# Create-Website

## Setting up a website from scratch using  
 * [HTTP](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol) and [optionally] [HTTPS](https://en.wikipedia.org/wiki/HTTPS)
 * [Nginx](https://nginx.org/en/docs/) as Web-Server
 * Namecheap for domain purchasing
 * [Letsencrypt](https://letsencrypt.org/) for SSL Certificate

## Objectives  
 * To set-up a website from scratch to without having to make any purchases [except for the server that will host your website], follow Steps 1 and 2. This will leave you with only an IP address and no FQDN [ie google.com, amazon.ca, etc] to access your website  
 * To set-up a website from scratch and also purchase and set up an FQDN, follow Steps 1, 2 and 3  
 * To set-up a website from scartch and also purchase and set up an FQDN as well as buy and configure the ssl certificate for allowing HTTPS requests, follow all the steps.  

[Basic Overall Steps to follow that I abstracted from to compose this README](http://www.howto-expert.com/how-to-get-https-setting-up-ssl-on-your-website/)

## Step 1: Install and set-up Nginx as the Web Server
Prereq questions you may have:  
 * [What is a Web-Server and its purpose?](https://en.wikipedia.org/wiki/Web_server) 
 * [What is ufw?](https://wiki.debian.org/Uncomplicated%20Firewall%20%28ufw%29)  

```shell
# Installing Nginx and UFW
sudo apt-get install -y nginx ufw

# ensuring that the server will allow HTTP requests
admin@ip-172-31-42-185:~$ sudo ufw allow "Nginx HTTP"
Rules updated
Rules updated (v6)


# ensuring that the server will allow ssh connections
admin@ip-172-31-42-185:~$ sudo ufw allow ssh
Rules updated
Rules updated (v6)

# Enable UFW on Server
admin@ip-172-31-78-96:~$ sudo systemctl start ufw
admin@ip-172-31-44-4:~$ sudo systemctl status ufw
● ufw.service - Uncomplicated firewall
   Loaded: loaded (/lib/systemd/system/ufw.service; enabled; vendor preset: enabled)
   Active: active (exited) since Mon 2018-03-05 05:32:31 UTC; 7s ago
     Docs: man:ufw(8)
  Process: 9583 ExecStart=/lib/ufw/ufw-init start quiet (code=exited, status=0/SUCCESS)
 Main PID: 9583 (code=exited, status=0/SUCCESS)

Mar 05 05:32:31 ip-172-31-44-4 systemd[1]: Starting Uncomplicated firewall...
Mar 05 05:32:31 ip-172-31-44-4 systemd[1]: Started Uncomplicated firewall.

admin@ip-172-31-44-4:~$ sudo ufw enable
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup

# ensuring that the rules we wanted are implemented
admin@ip-172-31-44-4:~$ sudo ufw status
Status: active

To                         Action      From
--                         ------      ----
Nginx HTTP                 ALLOW       Anywhere                  
22/tcp                     ALLOW       Anywhere                  
Nginx HTTP (v6)            ALLOW       Anywhere (v6)             
22/tcp (v6)                ALLOW       Anywhere (v6)                      

# determining status of nginx
admin@ip-172-31-44-4:~$ systemctl status nginx
● nginx.service - A high performance web server and a reverse proxy server
   Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
   Active: active (running) since Mon 2018-03-05 05:31:36 UTC; 2min 9s ago
     Docs: man:nginx(8)
  Process: 9520 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
  Process: 9518 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
 Main PID: 9522 (nginx)
    Tasks: 2 (limit: 4915)
   CGroup: /system.slice/nginx.service
           ├─9522 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
           └─9523 nginx: worker process

Mar 05 05:31:36 ip-172-31-44-4 systemd[1]: Starting A high performance web server and a reverse proxy server...
Mar 05 05:31:36 ip-172-31-44-4 systemd[1]: nginx.service: Failed to read PID from file /run/nginx.pid: Invalid argument
Mar 05 05:31:36 ip-172-31-44-4 systemd[1]: Started A high performance web server and a reverse proxy server.



# starting nginx program if it is not already started
sudo systemctl start nginx

# restart the nginx program if it is already started and you need to make new changes take effect
sudo systemctl restart nginx
```

### Step 1.a: Configuring Amazon AWS Security Group
*Only Applicable if you are using AWS*  
  
Please note that AWS provides an extra layer of protection for any instances[aka VMs] that you spin up. To modify the protection to suit your needs, you will need to modify the *Security Group* that has been assigned to the instance you are setting up the website on.  
  
  1.a.I. [Go to AWS Console](https://console.aws.amazon.com/ec2)  

  1.a.II. Select the Instance you are setting the website on ![Selecting Instance](Picture%201.png)  

  1.a.III. Click on the Security Group indicated on the box ![Selecting Security Group](Picture%202.png)  
  
  1.a.IV. Make sure that the rules listed under the "Inbound" Tabs include the ones in the picture here. If not, please add them exactly as shown. ![Inbound Rules](Picture%203.jpg)  

## Step 2: Configuring Nginx  
You will need to edit a file called the Nginx virtual hosts file. We will be editing this file in order to instruct nginx as to  
 * What is the folder that contains the index.html that we want to be displayed to the end-user
 * What is the port to listen on, port 80 is for HTTP and port 443 is for HTTPS. In this step, we will set up only port 80 for HTTPS

To determine other fun stuff you can do with the nginx virtual hosts file, feel free to refer to the [offical nginx documentation](https://nginx.org/en/docs/)    

```shell
sudo vim /etc/nginx/sites-available/default #Editing the Nginx virtual hosts file

# if the above file does not exist, run the following command to determine where the file has been installed on your machine
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
![Public IP/DNS Name](Picture%204.jpg)  
  
## Step 3: Buying a domain
This section just requires you to buy a FQDN [Fully Qualified Domain Name] such as google.com, amazon.ca, reddit.com, etc.  
Two popular places to buy a domain name from are Namecheap and GoDaddy. More options are available [here](https://en.wikipedia.org/wiki/Category:Domain_registrars) and by googling "buy domain name"  
  
  Step 3.a: Looking for available domain
  
  Step 3.b: Purchase the domain and any add-ons you may want.  
  
  Step 3.c: Register the IP of your instance with the domain name you just purchased.  
*Please note that after this step, it may take a while for it to work as you will have to wait for [1] Namecheap to register the I.P. address that you gave it with the domain you purchased as well as [2] waiting for the DNS that your computer queries to determine the IP address associated with a domain to be updated to reflect the change. Step 2 may take a while [up to an hour] just based on how the DNS that your computer qwueries has been set-up to refresh its cache.*  
  
Here is the bare minimum you need to configure to be able to access your new website via NameCheap
![NameCheap DNS Configuration](Picture%205.jpg)


## Step 4: Obtaining an SSL Certificate using Letsencrypt
*You will need to know the webroot for your site, the webroot is the directory that houses all the files and folders that are serviced to the web server [which in our case, is Nginx]. If you are unsure, please refer to the file path indicate on the nginx config file we modified earlier on in Step 2 that starts with the word "root"*
Quick Way to determine the root:

```shell
cat /etc/nginx/sites-available/default  | grep root
```


[Instructions available here](https://certbot.eff.org/)

Known Issues and how to resolve them:
```shell
admin@ip-172-31-44-4:~$ sudo apt-get install python-certbot-nginx -t stretch-backports
Reading package lists... Done
E: The value 'stretch-backports' is invalid for APT::Default-Release as such a release is not available in the sources
```
Steps to Resolve:
```shell
sudo bash -c 'echo "deb http://ftp.debian.org/debian stretch-backports main" >> /etc/apt/sources.list'
sudo apt-get update
```
  
## Step 5: Configuring ufw to allow HTTPS requests  
  
```shell
sudo ufw allow "Nginx HTTPS"
```

## Step 6: Configure the AWS Security Group to allow HTTPS requests
![AWS Security Group Configuration](Picture%207.jpg)

## Step 6: Configuring Nginx to redirect HTTP to HTTPS
[Reference Page for Setting up HTTPS](https://www.digicert.com/csr-ssl-installation/nginx-openssl.htm)
[Reference Page for redirecting HTTP to HTTPS](https://www.digitalocean.com/community/questions/best-way-to-configure-nginx-ssl-force-http-to-redirect-to-https-force-www-to-non-www-on-serverpilot-free-plan-by-using-nginx-configuration-file-only)
  
### Commands
  
You will need to edit a file called the Nginx virtual hosts file. We will be editing this file in order to instruct nginx as to  
 * What is the port to listen on, port 80 is for HTTP and port 443 is for HTTPS. In this step, we will use both as we will setup the server to automatically redirect any HTTP requests to HTTPS
 * Specifying the locations of the ssl_certificate and the ssl_certificate_key [this is needed in order to allow the server to accept HTTPS requests]  
  
To determine other fun stuff you can do with the nginx virtual hosts file, feel free to refer to the [offical nginx documentation]()  

```shell
sudo vim /etc/nginx/sites-available/default #Editing the Nginx virtual hosts file

##if the above file does not exist, run the following command to determine where the file has been installed on your machine
sudo find / -name sites-available
```
```
server {
  listen 80 default_server;
  listen [::]:80 default_server;
  server_name _;
  return 301 https://jasononline.ca$request_uri;
}

server {
  listen 443 default_server;
  listen [::];443 default_server;
  root /path/to/folder/that/contains/the/index
  index index.html index.htm index.nginx-debian.html;

  ssl  on;
  ssl_certificate  /etc/ssl/jasononline_ca.crt;
  ssl_certificate_key  /etc/ssl/jasononline-private-key.pm;
  server_name _;
  location / {
    try_files $uri $uri/ =404;
  }
}
```
```shell
sudo systemctl restart nginx #restarting nginx to allow changes to take effect
systemctl status nginx #checking status of nginx
```
[Nginx Reference Page](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-16-04)  
UFW Reference Pages  
 * [Ubuntu](https://help.ubuntu.com/community/UFW)  
 * [Debian](https://wiki.debian.org/Uncomplicated%20Firewall%20%28ufw%29)  