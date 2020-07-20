### Start with a basic more secure server
This assumes a Digital Ocean server/droplet, but should work on most Linux.  
login as root:  
```ssh root@your_server_ip```  
add new user:  
```adduser sammy```  
root priviledges:  
```usermod -aG sudo sammy```  
transfer ssh key stuffs:  
```rsync --archive --chown=sammy:sammy ~/.ssh /home/sammy```  
open new terminal window and ssh into server again without closing root login terminal:  
```ssh sammy@your_server_ip```  
make sure openssh is allowed through firewall  
```sudo ufw allow OpenSSH```  
```sudo ufw enable```  
```sudo apt update && sudo apt upgrade -y```  
```sudo nano /etc/ssh/sshd_config```  
Find:  
PermitRootLogin  
Make sure it is set to:  
```PermitRootLogin: no```  
Also find:  
PasswordAuthentication  
Make sure it is set to:  
```PasswordAuthentication: no```  
Restart SSH:  
```sudo service ssh restart```  
  
### Rename server name  
If you don't like the computer/server name:  
Type the following command to edit /etc/hostname using nano or vi text editor:  
```sudo nano /etc/hostname```  
Delete the old name and setup new name.  
Next Edit the /etc/hosts file:  
```sudo nano /etc/hosts```  
Replace any occurrence of the existing computer name with your new one.  
Reboot the system to changes take effect:  
```sudo reboot```  

### Set Nano text editor tab size
```nano /etc/nanorc```  
Use ```ctrl + w``` for where is to find ```#set tabsize 8```  
Change it to:  
```set tabsize 4```  
Also if preferred:
```set tabstospaces```  

### Install NginX web server  
NginX install:  
```sudo apt update && sudo apt install nginx -y```  
To see available NginX application profiles:  
```sudo ufw app list```  
Later, we will want to only allow HTTPS. For now allow full:  
```sudo ufw allow 'Nginx FULL'```  
We will worry about setting up LetsEncrypt Certbot for HTTPS shortly  
To see the updated UFW status:  
```sudo ufw status```  
To see the status of Nginx:  
```systemctl status nginx```  
To stop the webserver:  
```sudo systemctl stop nginx```  
To start the web server:  
```sudo systemctl start nginx```  
To restart the web server:  
```sudo systemctl restart nginx```  
To reload Nginx after configuration changes:  
```sudo systemctl reload nginx```  
To disable default automatic Nginx loading on server reboot:  
```sudo systemctl disable nginx```  
To enable default automatic Nginx loading on server reboot:   
```sudo systemctl enable nginx```  
  
### Make an NginX server block for your Website  
Nginx server blocks:  
```sudo mkdir -p /var/www/your_domain/public```  
Assign ownership of the directory:  
```sudo chown -R $USER:$USER /var/www/your_domain/public```  
Allow owner to read, write, and execute while restricting others:  
```sudo chmod -R 755 /var/www/your_domain```  
Create a sample page for demo purpose:  
```nano /var/www/your_domain/html/index.html```  
Paste this in:  
```
<html>
    <head>
        <title>Welcome to your_domain!</title>
    </head>
    <body>
        <h1>Success!  The your_domain server block is working!</h1>
    </body>
</html>
```
  
Create Nginx server block for new web page(virutal host in Apache speak):  
```sudo nano /etc/nginx/sites-available/your_domain```  
Paste in (PHP stuff added in for good measure):  
```
server {

        root /var/www/cordev.xyz/public;
        index index.html index.php;

        server_name cordev.xyz www.cordev.xyz;

        listen 443 ssl; #managed by Certbot

        # RSA certificate
        ssl_certificate /etc/letsencrypt/live/cordev.xyz/fullchain.pem; # managed by Certbot
        ssl_certificate_key /etc/letsencrypt/live/cordev.xyz/privkey.pem; # managed by Certbot

        include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot

        location / {
                try_files $uri $uri/ =404;
        }

        location ~ \.php$ {
                include snippets/fastcgi-php.conf;
                fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
        }

        location ~ /\.ht {
                deny all;
        }

}
```
  
Create system link for live sites:  
```sudo ln -s /etc/nginx/sites-available/your_domain /etc/nginx/sites-enabled/```  
Fix possible error:  
```sudo nano /etc/nginx/nginx.conf```  
Find:  
#server_names_hash_bucket_size 64;  
Change to:  
```server_names_hash_bucket_size 64;```  
Test to make sure there aren't errors:  
```sudo nginx -t```  
Restart webserver to enable changes:  
```sudo systemctl restart nginx```  
  
### Install PHP and MySQL
For ease of use, it's okay to install MySQL on the same server. However, it would be considered best practice to have a MySQL restricted to its own 
server and accessed remotely.  
Install MySQL:  
```sudo apt install mysql-server```  
Secure MySQL:  
```mysql_secure_installation```  
Install PHP:  
```sudo apt install php-fpm php-mysql```  
  
### Install LetsEncrypt for HTTPS SSL cert  
Assuming you have your domains, name-servers, cname DNS all set up it's time to install LetsEncrypt:  
```sudo apt install certbot python3-certbot-nginx```  
Install certs (-d = domain per each domain, including cname's):  
```sudo certbot --nginx -d example.com -d www.example.com```  
If you choose to set http traffic to redirect to https, then Certbot will configure your Nginx server block similar to:  
```
server {

        root /var/www/cordev.xyz/public;
        index index.html index.php;

        server_name cordev.xyz www.cordev.xyz;

        listen 443 ssl; #managed by Certbot

        # RSA certificate
        ssl_certificate /etc/letsencrypt/live/cordev.xyz/fullchain.pem; # managed by Certbot
        ssl_certificate_key /etc/letsencrypt/live/cordev.xyz/privkey.pem; # managed by Certbot

        include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot

        location / {
                try_files $uri $uri/ =404;
        }

        location ~ \.php$ {
                include snippets/fastcgi-php.conf;
                fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
        }

        location ~ /\.ht {
                deny all;
        }

}

server {

        if ($host = www.cordev.xyz) {   
                return 301 https://$host$request_uri;
        } # managed by Certbot

        if ($host = cordev.xyz) {
                return 301 https://$host$request_uri;
        } # managed by Certbot

        listen 80;
        listen [::]:80;

        server_name cordev.xyz www.cordev.xyz;
        return 404; # managed by Certbot

}
```
### Automatically Renew Let’s Encrypt Certificates
Open the crontab file:  
```crontab -e```  
Add the following at the bottom:  
```0 12 * * * /usr/bin/certbot renew --quiet```  
  
### Useful reading/sources:  
streamlined from:  
https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-18-04  
  
https://www.digitalocean.com/community/questions/how-can-i-disable-ssh-login-for-a-root-user-i-am-the-account-owner  
https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-20-04  
https://www.digitalocean.com/community/tutorials/how-to-install-linux-nginx-mysql-php-lemp-stack-on-ubuntu-20-04  
https://www.nginx.com/blog/using-free-ssltls-certificates-from-lets-encrypt-with-nginx/  
https://www.nano-editor.org/dist/latest/cheatsheet.html

### Further reading  
https://cloud-init.io/  