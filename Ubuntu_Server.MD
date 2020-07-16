
make digital ocean server
login as root:
ssh root@your_server_ip
add new user:
adduser sammy
root priviledges:
usermod -aG sudo sammy
transfer ssh key stuffs:
rsync --archive --chown=sammy:sammy ~/.ssh /home/sammy
open new terminal window and ssh into server again without closing root login terminal:
ssh sammy@your_server_ip
make sure openssh is allowed through firewall
sudo ufw allow OpenSSH
sudo ufw enable
sudo apt update && sudo apt upgrade -y
sudo nano /etc/ssh/sshd_config: 
Find:
PermitRootLogin
Make sure it is set to:
PermitRootLogin: no
Also find:
PasswordAuthentication
Make sure it is set to:
PasswordAuthentication: no
Restart SSH:
sudo service ssh restart

If you don't like the computer/server name:
Type the following command to edit /etc/hostname using nano or vi text editor:
sudo nano /etc/hostname
Delete the old name and setup new name.
Next Edit the /etc/hosts file:
sudo nano /etc/hosts
Replace any occurrence of the existing computer name with your new one.
Reboot the system to changes take effect:
sudo reboot


NginX install:
sudo apt update && sudo apt install nginx -y
To see available NginX application profiles:
sudo ufw app list
Later, we will want to only allow HTTPS. For now allow full:
sudo ufw allow 'Nginx FULL'
We will worry about setting up LetsEncrypt Certbot for HTTPS shortly
To see the updated UFW status:
sudo ufw status
To see the status of Nginx:
systemctl status nginx
To stop the webserver:
sudo systemctl stop nginx
To start the web server:
sudo systemctl start nginx
To restart the web server:
sudo systemctl restart nginx
To reload Nginx after configuration changes:
sudo systemctl reload nginx
To disable default automatic Nginx loading on server reboot:
sudo systemctl disable nginx
To enable default automatic Nginx loading on server reboot: 
sudo systemctl enable nginx

Nginx server blocks:
sudo mkdir -p /var/www/your_domain/public
Assign ownership of the directory:
sudo chown -R $USER:$USER /var/www/your_domain/public
Allow owner to read, write, and execute while restricting others:
sudo chmod -R 755 /var/www/your_domain
Create a sample page for demo purpose:
nano /var/www/your_domain/html/index.html
Paste this in:
<html>
    <head>
        <title>Welcome to your_domain!</title>
    </head>
    <body>
        <h1>Success!  The your_domain server block is working!</h1>
    </body>
</html>

Create Nginx server block for new web page(virutal host in Apache speak):
sudo nano /etc/nginx/sites-available/your_domain
Paste in (PHP stuff added in for good measure):
server {
        listen 443;
        listen [::]:443;

        root /var/www/cordev.xyz/public;
        index index.html index.php;

        server_name cordev www.cordev;

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

Create system link for live sites:
sudo ln -s /etc/nginx/sites-available/your_domain /etc/nginx/sites-enabled/
Fix possible error:
sudo nano /etc/nginx/nginx.conf
Find: 
#server_names_hash_bucket_size 64;
Change to:
server_names_hash_bucket_size 64;
Test to make sure there aren't errors:
sudo nginx -t
Restart webserver to enable changes:
sudo systemctl restart nginx
For ease of use, it's okay to install MySQL on the same server. However, it would be considered best practice to have a MySQL restricted to its own 
server and accessed remotely.
Install MySQL:
sudo apt install mysql-server
Install PHP:
sudo apt install php-fpm php-mysql

Assuming you have your domains, name-servers, cname DNS all set up it's time to install LetsEncrypt:
sudo apt install certbot python3-certbot-nginx
Install certs (-d = domain per each domain, including cname's):
sudo certbot --nginx -d cordev.xyz -d www.cordev.xyz




Useful reading/sources:
streamlined from: https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-18-04
https://www.digitalocean.com/community/questions/how-can-i-disable-ssh-login-for-a-root-user-i-am-the-account-owner
https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-20-04
https://www.digitalocean.com/community/tutorials/how-to-install-linux-nginx-mysql-php-lemp-stack-on-ubuntu-20-04




Further reading
https://cloud-init.io/




sudo certbot --nginx -d cordev.xyz -d www.cordev.xyz -d appdemo001.cordev.xyz -d appdemo002.cordev.xyz -d appdemo003.cordev.xyz -d appdemo004.cordev.xyz -d appdemo005.cordev.xyz -d appdemo006.cordev.xyz -d appdemo007.cordev.xyz -d appdemo008.cordev.xyz -d appdemo009.cordev.xyz -d appdemo010.cordev.xyz -d appdemo100.cordev.xyz -d appdemo200.cordev.xyz -d appdemo300.cordev.xyz -d appdemo400.cordev.xyz -d appdemo500.cordev.xyz -d appdemo600.cordev.xyz -d appdemo700.cordev.xyz -d appdemo800.cordev.xyz -d appdemo900.cordev.xyz -d appdemo999.cordev.xyz 