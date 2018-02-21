# Setting up a website from scratch to use HTTPS with Nginx as a Web-server, Namecheap for domain purchasing and letsencrypt for ssl certificate
  
 * To set-up a website from scratch to without having to make any purchases [except for the server that will host your website], follow Steps 1 and 2. This will leave you with only an IP address and not FQDN [ie google.com, amazon.ca, etc] to access your website  
 * To set-up a website from scratch and also purcahse and set up an FQDN, follow Steps 1, 2 and 3  
 * To set-up a website from scartch and also purchase and set up an FQDN as well as buy and configure the ssl certificate for allowing HTTPS requests, follow all the steps.  

[Basic Overall Steps to follow that I abstracted from to compose this README](http://www.howto-expert.com/how-to-get-https-setting-up-ssl-on-your-website/)

## Step 1: Install and set-up Nginx as the Web Server
Prereq questions you may have:  
 * [What is a Web-Server and its purpose?](https://en.wikipedia.org/wiki/Web_server) 
 * [What is ufw?](https://wiki.debian.org/Uncomplicated%20Firewall%20%28ufw%29)  

```shell
apt-get install -y nginx #installing nginx

# ensuring that the server will allow HTTP requests
sudo ufw allow "Nginx HTTP"

# ensuring that the server will allow ssh connections
sudo ufw allow ssh

# ensuring that the rules we wanted are implemented
sudo ufw status

# determining status of nginx
systemctl status nginx

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
  
  1.a.IV. Make sure that the rules listed under the "Inbound" Tabs include the ones in the picture here. If not, please add them exactly as shown. ![Inbound Rules](Picture%203.png)  

## Step 2: Configuring Nginx  
You will need to edit a file called the Nginx virtual hosts file. We will be editing this file in order to instruct nginx as to  
 * What is the folder that contains our index.html that we want to be displayed to the end-user
 * What is the port to listen on, port 80 is for HTTP and port 443 is for HTTPS. In this step, we will set up only port 80 for HTTPS

To determine other fun stuff you can do with the nginx virtual hosts file, feel free to refer to the [offical nginx documentation](https://nginx.org/en/docs/)    

```shell
sudo vim /etc/nginx/sites-available/default #Editing the Nginx virtual hosts file

# if the above file does not exist, run the following command to determine where the file has been installed on your machine
sudo find / -name sites-available
```
```
server {
  listen 80 default_server;
  listen [::]:80 default_server;
  server_name _;

  root /path/to/folder/that/contains/the/index

  index index.html index.htm index.nginx-debian.html;

  location / {
    try_files $uri $uri/ =404;
  }
}
}
```
```shell
sudo systemctl restart nginx #restarting nginx to allow changes to take effect
systemctl status nginx #checking status of nginx
```

  
## Step 3: Buying a domain
This section just requires you to buy a FQDN [Fully Qualified Domain Name] such as google.com, amazon.ca, reddit.com, etc.  
Two popular places to buy a domain name from are Namecheap and GoDaddy. More options are available [here](https://en.wikipedia.org/wiki/Category:Domain_registrars) and by googling "buy domain name"  
  
### Step 3.a: Looking for available domain
  
### Step 3.b: Purchase the domain and any add-ons you may want.  
  
### Step 3.c: Register the IP of your instance with the domain name you just purchased.  
*Please note that after this step, it may take a while for it to work as you will have to wait for [1] Namecheap to register the I.P. address that you gave it with the domain you purchased as well as [2] waiting for the DNS that your computer queries to determine the IP address associated with a domain to be updated to reflect the change. Step 2 may take a while [up to an hour] just based on how the DNS that your computer qwueries has been set-up to refresh its cache.*  
  


## Step 4: Generate a CSR
Steps were pulled off of this site: [Namecheap Documentation](https://www.namecheap.com/support/knowledgebase/article.aspx/467/67/how-do-i-generate-a-csr-code)  
Specifically from this page: [NameCheap Documentation for AWS Instances](https://www.namecheap.com/support/knowledgebase/article.aspx/9592/0/aws)
```shell
sudo apt-get install -y openssl
openssl genrsa 2048 > jasononline-private-key.pem
openssl req -new -key jasononline-private-key.pem -out csr.pem
```

## Step 5: Buy a Certificate
You will need to shop for a certificate online, a common choice is namecheap, you can look at prices here: [https://www.namecheap.com/security/ssl-certificates.aspx](https://www.namecheap.com/security/ssl-certificates.aspx)  
  
## Step 6: Activating/Installing the Certificate
[Instructions from Namecheap for how to complete the domain control validation (DCV) for my SSL certificate](https://www.namecheap.com/support/knowledgebase/article.aspx/9637/68/how-can-i-complete-the-domain-control-validation-dcv-for-my-ssl-certificate)  
  
## Step 7: Configuring ufw to allow HTTPS requests  
  
```shell
sudo ufw allow "Nginx HTTPS"
```

## Step 8: Configuring Nginx to redirect HTTP to HTTPS
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