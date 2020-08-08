difference between openJDK and Java hotspot
ans : both are similar 
except Oracle JVM has a few additional commercial features, mainly, 
Java Flight Recorder, Application Class Data Sharing and Cooperative Memory Management.

$ if [[ $(java -version 2>&1) == *"OpenJDK"* ]]; then echo ok; else echo 'not ok'; fi
not ok
$ java -version
java version "1.8.0_201"
Java(TM) SE Runtime Environment (build 1.8.0_201-b09)
Java HotSpot(TM) 64-Bit Server VM (build 25.201-b09, mixed mode)

how to check ubuntu version : lsb_release -a
$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu Groovy Gorilla (development branch)
Release:	20.10
Codename:	groovy

https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-18-04

install ngnix for newer version :
1. switch to root and then run bellow commands.

nginx=stable # use nginx=development for latest development version
add-apt-repository ppa:nginx/$nginx
apt-get update
apt-get install nginx

once installation done check http://localhost --> if this gives web page then installation is completed
path where installtion done : 
/etc/nginx$ ls
conf.d        fastcgi_params  koi-win     modules-available  nginx.conf    scgi_params      sites-enabled  uwsgi_params
fastcgi.conf  koi-utf         mime.types  modules-enabled    proxy_params  sites-available  snippets       win-utf

firewall setting check for ngnix : 
sudo ufw app list

enable : sudo ufw allow 'Nginx HTTP'
verify the change : sudo ufw status

### ngnix administration command : 
1. check the web server status:
systemctl status nginx
2. To stop your web server, type: 
sudo systemctl stop nginx
3. To start the web server when it is stopped, type: 
sudo systemctl start nginx
4. To stop and then start the service again, type: 
sudo systemctl restart nginx
5. If you are simply making configuration changes, Nginx can often reload without dropping connections. To do this, type: 
sudo systemctl reload nginx
6. By default, Nginx is configured to start automatically when the server boots. If this is not what you want, you can disable this behavior by typing:
sudo systemctl disable nginx
7. To re-enable the service to start up at boot, you can type:
sudo systemctl enable nginx

----------------------------------------------------------------------------------------------------------------------

https://www.digitalocean.com/community/tutorials/how-to-install-jenkins-on-ubuntu-18-04
1. install Jenkins : 
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo vi /etc/apt/sources.list , and add below line to the end of the file :
deb https://pkg.jenkins.io/debian-stable binary/
sudo apt-get update
sudo apt-get install jenkins

2. Chnage the port number of the jenkins : 
file to edit the HTTP_PORT : 
/etc/default/jenkins

jenkins server administration : 
sudo systemctl start/stop/restart/status jenkins

allow firewall to open port 9001 which has been assigned by user to open jenkins . 

/etc/default$ sudo ufw status
Status: inactive
/etc/default$ sudo ufw allow 9001
Rules updated
Rules updated (v6)
/etc/default$ sudo ufw status
Status: inactive
/etc/default$ sudo ufw allow OpenSSH
Rules updated
Rules updated (v6)
/etc/default$ sudo ufw status
Status: inactive
/etc/default$ sudo ufw enable
Firewall is active and enabled on system startup
/etc/default$ sudo ufw status
Status: active

To                         Action      From
--                         ------      ----
8080                       ALLOW       Anywhere                  
Nginx HTTP                 ALLOW       Anywhere                  
9001                       ALLOW       Anywhere                  
OpenSSH                    ALLOW       Anywhere                  
8080 (v6)                  ALLOW       Anywhere (v6)             
Nginx HTTP (v6)            ALLOW       Anywhere (v6)             
9001 (v6)                  ALLOW       Anywhere (v6)             
OpenSSH (v6)               ALLOW       Anywhere (v6)             




initially get the admin password : 
sudo cat /var/lib/jenkins/secrets/initialAdminPassword

once plugin installation completed provide username annd login password
http://localhost:9001/JENKINS
