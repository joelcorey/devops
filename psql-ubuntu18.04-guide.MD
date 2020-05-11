## psql (postgres) Ubuntu 18.04 install guide
## This guide will mirror the Thinkful Mac OS installation guide as closely as possible

All reference material links here:  
https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-ubuntu-16-04  
https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-18-04  
https://www.databaselabs.io/blog/img/Postgres-Cheat-Sheet.pdf  
https://courses.thinkful.com/ei-node-postgres-v1/checkpoint/11  
https://courses.thinkful.com/ei-node-postgres-v1/checkpoint/12  
https://askubuntu.com/questions/32730/how-to-remove-postgres-from-my-installation  

Install - Update repository sources and install necessary Postgres stuff:  
```sudo apt update && sudo apt install postgresql postgresql-contrib -y```  
  
To switch to the postgres system user and open postgres:  
```sudo -u postgres psql```  
  
Get config file locations  
```sudo -u postgres psql -c 'SHOW config_file'```  
  
Change timezone to UTC:  
Find timezone = and set it to 'UTC' in:  
```sudo nano /etc/postgresql/11/main/postgresql.conf```  
For a U.S./American West Coast time (UTC -08:00) fine the line labelled:
```timezone = 'Etc/UTC'```  
And change it to:  
```timezone = 'America/Los Angeles'```  
For a complete list of timezones perform the following SQL query:  
```SELECT name, abbrev, utc_offset, is_dst FROM pg_timezone_names;'''  
  
At least to start with, Postgres on Linux uses an identity based security system. You need a user titled 'postgres' in 
order to get things up and running. If your user name is 'derp', then make a PSQL user named 'derp'.
Run this command:  
```adduser```  
Also: remember the password you set for this.  
And to switch to this user:  
```su postgres```  
Or to run commands from your normal account without switching:  
```sudo -u postgres createuser --interactive```  
  
Use the above command to create a Postgres role that matches the user for your system. This matches the 
Postgres Linux security model.  
  
Once you have a Postgres role name after your default user on Linux, you can create a database. 
  
After making changes to a config file for Postgres, use systemctl to restart the service:  
```sudo systemctl restart postgresql```  

Forgot to add a password when creating a user? No problem.
Assuming you have a PSQL user the same as your system user:
``` psql <databasename>```    
```ALTER ROLE <psql-username> WITH PASSWORD <password>```  
  
Postgres does not allow remote connections by default. Edit two files to enable:  
```sudo nano /etc/postgresql/10/main/postgresql.conf```  
and find:
```#listen_addresses = 'localhost'```
Notice that this is commented out as it is default behaviour. Leave this commented out and above it put:  
```listen_addresses = '*'```  
And then in:  
```sudo nano /etc/postgresql/10/main/pg_hba.conf```  
Add at the bottom of the file:  
```host    all             all             0.0.0.0/0               md5```  
```host    all             all             ::0/0                   md5```
And since we changed config files:
```sudo systemctl restart postgresql```  
  
If you have UFW (firewall) active then:  
```sudo ufw allow 5432/tcp```  
  
(Not required for quick start) We need to add pg_ctl to the system path variable. We can do this in 
the ~/.profile file:  
```nano ~/.bashrc.```  
  
You should see something similar to:  
```export PATH=~/.some-directory-here:/some-other-directory:$PATH```  
  
(Not required for quick start) We want to add the pg_ctl to PATH like this:  
```export PATH=/usr/lib/postgresql/11/bin:$PATH```  
  
Note that the 11 for the postgresql version may change, and you will need to match this to the installed 
postgres version.  
  
Get the postgres version like this:  
```psql --version```  
  
To get the corrent data directory that postgres is using:  
```sudo -u postgres psql```  
and:  
```SHOW data_directory;```  
This should output something very similar to:  
```/var/lib/postgresql/11/main```  
  
To change the data directory:  
```sudo nano /etc/postgresql/11/main/postgresql.conf```  
  
In order to use pg_ctl we have to make a new var in ~/.bashrc called PGDATA:  
```export PGDATA=/var/lib/postgresql/11/main```  
And to reload it:  
```. ~/.bashrc```  
  
Typing  
```CREATE ROLE ...```  
inside of the psql prompt is the same as typing  
```createuser -Pw --interactive```  
outside of it.  
  
To create a database, and due to the how Linux/Postgres is setup by default, we have to switch user to either the 
postgres user or to a linux user that also has a role defined through PostgresQL:  
```sudo su postgres```  
and then:  
```createdb <database_name>```  
  
To drop a user (actually a role):  
```DROP ROLE <name>```  
and if the user/role name has a "-" in it surround user name with double quotes.  
  
display roles  
```\du```  
questions  
```\q```  
filename - import sql script  
```\i```  
dbname - connect to database name  
```\c```  
