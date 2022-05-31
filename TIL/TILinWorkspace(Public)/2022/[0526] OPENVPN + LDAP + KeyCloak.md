Settings in Keycloak



Groups / Role Mappings :  
1.	Default-roles-demo : assign the roles by default.  
2.	Offline-access : e.g. periodic backup of some data  
3.	Uma-authorization : User-management-Access authorization (usually it’s for admin)  

* Usually the application save the offline tokens into database usually also using it to manually retrieve new access token 
from keycloak server. Also, the offline token will never expire and is not subject of SSO Session Idle timeout. 
* The offline token is valid even after a user logout or server restart. However by default, 
* the user do need to use the offline token for a refresh token action at least once per 30 days.
* (= by the setting : Offline Session Idle timeout – it can be changed in the administration console in the tokens -)
  
0. Prerequisites  
      0-1.	Install Keycloak  
      wget https://github.com/keycloak/keycloak/releases/download/17.0.1/keycloak-17.0.1.tar.gz  
      0-2.	Unzip keycloak  
      tar -zxf keycloak-17.0.1.tar.gz  
      0-3.	Install Open JDK 11  
      0-4.	Fix resolv.conf  
      sudo vi /etc/resolv.conf  
      *change the nameserver configuration to -> nameserver 8.8.8.8*  
      sudo apt update  
      sudo apt install openjdk-11-jre-headless  
      java -version  
      0-5.	Configure hostname of keycloak  
      cd ~/keycloak-17.0.1/conf  
      vi keycloak.conf  
      *add configuration as follows:  
      hostname-strict=false  
      hostname-strict-https=false  
      http-enabled=true  
      http-port=5010

0-6.	Start Keycloak  
cd ~/keycloak-17.0.1/  
bin/kc.sh start  
0-7.	Create an initial admin user  
selabdev / password  
0-8.	Build with postgres  
bin/kc.sh build --db postgres  
0-9.	Configure conection information with postgres  
*http-relative-path=/auth  
db=postgres  
db-username=postgres  
db-password=password  
db-url=jdbc:postgresql://localhost:5432/postgres*  
0-10.	Install Postgresql  
0-11.	Add configs for DNS  
sudo vi /etc/network/interfaces  
add *dns-nameservers 8.8.8.8 8.8.4.4*  
sudo vi /etc/resolv.conf  
*nameserver 8.8.8.8  
Nameserver 8.8. 4.4*  
sudo apt update  
sudo apt install postgresql postgresql-contrib  
0-12.	Change access information for postgres  
sudo find / -name pg_hba.conf  
cd /etc/postgresql/10/main/  
sudo vi pg_hba.conf  
change to local connections to trust from md5/peer  
  
0-13.	Check the connection of postgres  
service postgresql status  
sudo service postgresql stop  
sudo service postgresql start  
0-14.	Create new server with postgresql  
  
  
  
0-15.	Create new db withb postgres  
  
0-16.	Execute psql  
sudo service postgresql start  
sudo -u postgres psql  
postgres=# \l  
  
0-17.	Restart Keycloak with postgres  
  
  
Success !  
1.	LDAP + KeyCloak  
      1-1.	Install LDAP  
      sudo apt update  
      sudo apt install slapd ldap-utils  
      *set up the development environment with this ref: https://med  ium.com/analytics-vidhya/install-openldap-with-phpldapadmin-on-ubuntu-9e56e57f741e*
      1-2.	Configure the openLDAP server post installation  
      Sudo dpkg-reconfigure slapd  
      1-3.	Configure LDAP clients  
      sudo vi /etc/ldap/ldap.conf  
      add  
      *TLS_REQCERT allow  
      BASE dc=selab,dc=cloud  
      URI ldap://localhost*  
      1-4.	Test the server  
      sudo service slapd status  
      sudo service slapd start  
      ldapsearch -x  
      1-5.	Check output  
  
1-6.	Install phpldapadmin  
sudo apt install phpldapadmin  
1-7.	Configure phpldapadmin  
sudo vi /etc/phpldapadmin/config.php  
1-8.	Set the timezone accordingly  
(find word with vim : type to ? { KEYWORD })  
modify to *config->custom->appearance[‘timezone’] = ‘Asia/Seoul’*;  
*servers->setValue('server','name',’{ CUSTOM-SERVERNAME }’)*  
*servers->setValue('server','host',’{ CUSTOM IP ADDRESS }’);*  
*config->custom->appearance['hide_template_warning'] = true;*  
*servers->setValue('login','anon_bind',false);*  
1-9.	Generate new User Federation with Keycloak  
  
  
  
2.	OpenVPN + LDAP  
      2-1. Install OpenVPN plugin for LDAP authentication  
      sudo apt install openvpn-auth-ldap  
3.	  
  