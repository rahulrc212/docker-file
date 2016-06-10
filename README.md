# docker-file

FROM ubuntu

MAINTAINER <TALSON THOMAS>
apt-get update && apt-get upgrade -y
sudo apt-get install curl unzip vim wget links -y

#Install LAMP Stack

apt-get install apache2 -y

apt-get install mysql-server php5-mysql -y

#To check the ip and mac address #ip addr show eth0 | grep inet | awk '{ print $2; }' | sed 's/\/.*$//'

#MySQL to create its database directory structure

mysql_install_db

start mysql start
#mysql_secure_installation -n \

#\ -y \ -y \ -y \ -y

apt-get install php5 libapache2-mod-php5 php5-mcrypt -y

#cat >> index.php | vim /etc/apache2/mods-enabled/dir.conf  
#<IfModule mod_dir.c>
#    DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
#</IfModule>


#Install Nagios

useradd nagios
groupadd nagcmd
usermod -a -G nagcmd nagios

apt-get update && apt-get upgrade

apt-get install build-essential libgd2-xpm-dev openssl libssl-dev xinetd apache2-utils -y

#Install Nagios Core

cd~ && curl -L -O https://assets.nagios.com/downloads/nagioscore/releases/nagios-4.1.1.tar.gz

tar xvf nagios-4.1.1.tar.gz 

# apt-get install postfix -y

cd /root/nagios-4*
./configure --with-nagios-group=nagios --with-command-group=nagcmd ##--with-mail=/usr/sbin/postfix	

make all			
make install
make install-commandmode
make install-init
make install-config
/usr/bin/install -c -m 644 sample-config/httpd.conf /etc/apache2/sites-available/nagios.conf
sudo usermod -G nagcmd www-data


#Install Nagios-Plugins

cd ~
curl -L -O http://nagios-plugins.org/download/nagios-plugins-2.1.1.tar.gz

tar xvf nagios-plugins-*.tar.gz

cd nagios-plugins-*

./configure --with-nagios-user=nagios --with-nagios-group=nagios --with-openssl

make

make install

#Install NRPE

cd ~
curl -L -O http://downloads.sourceforge.net/project/nagios/nrpe-2.x/nrpe-2.15/nrpe-2.15.tar.gz

tar xvf nrpe-*.tar.gz

cd nrpe-*

./configure --enable-command-args --with-nagios-user=nagios --with-nagios-group=nagios --with-ssl=/usr/bin/openssl --with-ssl-lib=/usr/lib/x86_64-linux-gnu

make all
sudo make install
sudo make install-xinetd
sudo make install-daemon-config

#Open the xinetd startup script in an editor:
# vim /etc/xinetd.d/nrpe
#Modify the only_from line by adding the private IP address of the your Nagios server to the end 
# only_from = 127.0.0.1 172.17.0.3
#service xinetd restart

#Organize Nagios Configuration
#vim /usr/local/nagios/etc/nagios.cfg
#uncomment the --> cfg_dir=/usr/local/nagios/etc/servers
#mkdir /usr/local/nagios/etc/servers

#Open the Nagios contacts configuration in your favorite text editor.
# vim /usr/local/nagios/etc/objects/contacts.cfg
# Find the email directive, and replace its value (the highlighted part) with your own email address

#Configure check_nrpe Command (Configure it using Nconf)
#vim /usr/local/nagios/etc/objects/commands.cfg
#define command{
#        command_name check_nrpe
#        command_line $USER1$/check_nrpe -H $HOSTADDRESS$ -c $ARG1$
#}
	

#Configure Apache

#Enable the Apache rewrite and cgi modules:
# sudo a2enmod rewrite
# sudo a2enmod cgi

#htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin

#If you create a user that is not named "nagiosadmin", you will need to edit /usr/local/nagios/etc/cgi.cfg and change all the "nagiosadmin" references to the user you created.

#Now create a symbolic link of nagios.conf to the sites-enabled directory:
#sudo ln -s /etc/apache2/sites-available/nagios.conf /etc/apache2/sites-enabled/

#service nagios start
#service apache2 restart

#To enable Nagios to start on server boot, run this command:
#sudo ln -s /etc/init.d/nagios /etc/rcS.d/S99nagios

#If you want to restrict the IP addresses that can access the Nagios web interface, you will want to edit the 
# sudo vim /etc/apache2/sites-available/nagios.conf
#Find and comment the following two lines by adding # symbols in front of them:
#Order allow,deny
#Allow from all

#Then uncomment the following lines, by deleting the # symbols, and add the IP addresses or ranges (space delimited) that you want to allow to in the Allow from line:
# Order deny,allow
# Deny from all
# Allow from 127.0.0.1


service nagios restart
service apache2 restart

#Install Nconf

wget https://sourceforge.net/projects/nconf/files/nconf/1.3.0-0/nconf-1.3.0-0.tgz

sudo tar xzvf nconf-1.3.0-0.tgz -C /var/www/

sudo chown -R www-data:www-data /var/www/nconf

create database "nconf" in mysql

Import file "create_database.sql" from nconf directory to nconf database

configure nconf using url/nconf

show path to nconf for nagios binary file location

set permission for nagios binary and nconf directory as www-data

chown -R www-data:www-data /var/www/nconf
chown www-data:www-data /usr/local/bin/nagios
