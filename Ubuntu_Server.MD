streamlined from: https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-18-04
https://www.digitalocean.com/community/questions/how-can-i-disable-ssh-login-for-a-root-user-i-am-the-account-owner

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