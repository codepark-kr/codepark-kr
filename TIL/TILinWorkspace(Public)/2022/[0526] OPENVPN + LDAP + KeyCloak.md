## OpenVPN Community Edition + LDAP + Keycloak
`2022.05.27. - `

## Introduction
사내 인프라 개선 프로젝트의 일부로, OpenVPN Community Edition + LDAP + Keycloak을 연동하고, 사용자 계정 생성 및 권한 설정을 부여한다. 

---

## Task
* [x] Keycloak 기본 설정
* [x] LDAP + Keycloak
* [ ] RSA, phpldapadmin 설정

---

## Details

### Task #1 Keycloak 기본 설정
![img.png](../../../Assets/images/ldap-image1.png)

Groups / Role Mappings :
1.	Default-roles-demo : assign the roles by default.
2.	Offline-access : e.g. periodic backup of some data
3.	Uma-authorization : User-management-Access authorization (usually it’s for admin)

* Usually the application save the offline tokens into database usually also using it to manually retreive new access token
* from keycloak server. Also, the offline token will never expire and is not subject of SSO Session Idle timeout. 
* The offline token is valid even after a user logout or server restart. However by default, 
* the user do need to use the offline token for a refresh token action at least once per 30 days.
* (= by the setting : Offline Session Idle timeout – it can be changed in the administration console in the tokens -)

1-1.	Install Keycloak  
`wget https://github.com/keycloak/keycloak/releases/download/17.0.1/keycloak-17.0.1.tar.gz`  

1-2.	Unzip keycloak  
`tar -zxf keycloak-17.0.1.tar.gz`  

1-3.	Install Open JDK 11  
`sudo apt install openjdk-11-jre-headless`  

1-4.	Fix resolv.conf  
`sudo vi /etc/resolv.conf`  
change the nameserver configuration to -> `nameserver 8.8.8.8`  
`sudo apt update`  
`java -version`  

1-5.	Configure hostname of keycloak  
`cd ~/keycloak-17.0.1/conf`  
`vi keycloak.conf`  
add configuration as follows:
```
hostname-strict=false  
hostname-strict-https=false  
http-enabled=true  
http-port=5010
```  

1-6.	Start Keycloak  
`cd ~/keycloak-17.0.1/`  
`bin/kc.sh start`  

1-7.	Create an initial admin user  
`selabdev / password`  

1-8.	Build with postgres  
`bin/kc.sh build --db postgres`  

1-9.	Configure connection information with postgres  
```
http-relative-path=/auth
db=postgres
db-username=postgres
db-password=password
db-url=jdbc:postgresql://localhost:5432/postgres
```  

1-10.	Install Postgresql  

1-11.	Add configs for DNS  
`sudo vi /etc/network/interfaces`  
add `dns-nameservers 8.8.8.8 8.8.4.4`  
`sudo vi /etc/resolv.conf`  
```
nameserver 8.8.8.8
nameserver 8.8.4.4
```  
`sudo apt update`  
`sudo apt install postgresql postgresql-contrib`  

1-12.	Change access information for postgres  
`sudo find / -name pg_hba.conf`  
`cd /etc/postgresql/10/main/`  
`sudo vi pg_hba.conf`  
change to local connections to `trust` from `md5/peer`  
![img_2.png](../../../Assets/images/img_2.png)  

1-13.	Check the connection of postgres  
`service postgresql status`  
`sudo service postgresql stop`  
`sudo service postgresql start`  

1-14.	Create new server with postgresql
![img_5.png](../../../Assets/images/img_5.png)  
![img_4.png](../../../Assets/images/img_4.png)  
![img_6.png](../../../Assets/images/img_6.png)  


1-15.	Create new db withb postgres  
![img_8.png](../../../Assets/images/img_8.png)  

1-16.	Execute psql  
```
sudo service postgresql start
sudo -u postgres psql
postgres=# \l
```  

![img_10.png](../../../Assets/images/img_10.png)  


1-17.	Restart Keycloak with postgres
![img_12.png](../../../Assets/images/img_12.png)  
![img_13.png](../../../Assets/images/img_13.png)  

---

### Task #2 LDAP + KeyCloak

2-1.	Install LDAP
```
sudo apt update
sudo apt install slapd ldap-utils
```  
set up the development environment with this ref:  
https://medium.com/analytics-vidhya/install-openldap-with-phpldapadmin-on-ubuntu-9e56e57f741e

2-2.	Configure the openLDAP server post installation  
`sudo dpkg-reconfigure slapd`  

2-3.	Configure LDAP clients  
`sudo vi /etc/ldap/ldap.conf`  
add to :  
```
TLS_REQCERT allow
BASE dc=selab,dc=cloud
URI ldap://localhost
```

2-4.	Test the server  
```
sudo service slapd status
sudo service slapd start
ldapsearch -x
```
2-5.	Check output  
![img_15.png](../../../Assets/images/img_15.png)  

2-6.	Install phpldapadmin  
`sudo apt install phpldapadmin`  

2-7.	Configure phpldapadmin  
`sudo vi /etc/phpldapadmin/config.php`  

2-8.	Set the timezone accordingly  
(find word with vim : type to ? { KEYWORD })  
modify to `config->custom->appearance[‘timezone’] = ‘Asia/Seoul’;`  
```
servers->setValue('server','name',’{ CUSTOM-SERVERNAME }’)
servers->setValue('server','host',’{ CUSTOM IP ADDRESS }’);
config->custom->appearance['hide_template_warning'] = true;
servers->setValue('login','anon_bind',false);
```

2-9.	Generate new User Federation with Keycloak
![img_18.png](../../../Assets/images/img_18.png)  
![img_17.png](../../../Assets/images/img_17.png)  

---

### Task #3: RSA + phpldapadmin

3-1. Install OpenVPN plugin for LDAP authentication  
`sudo apt install openvpn-auth-ldap`    

3-2. Install Easy-RSA dependecies  
```
wget https://github.com/OpenVPN/easy-rsa/archive/3.0.1.tar.gz
tar xzvf 3.0.1.tar.gz
rm 3.0.1.tar.gz
```

3-3. Customize the easy-RSA Configuration  
```
cd easy-rsa-3.0.1/easyrsa3
cp vars.example vars
echo 'set_var EASYRSA_DN "org"' >> vars
echo 'set_var EASYRSA_KEY_SIZE 4096' >> vars
echo 'set_var EASYRSA_REQ_COUNTRY "DE"' >> vars
echo 'set_var EASYRSA_REQ_PROVINCE "Berlin"' >> vars
echo 'set_var EASYRSA_REQ_CITY "Berlin"' >> vars
echo 'set_var EASYRSA_REQ_ORG "BigchainDB GmbH"' >> vars
echo 'set_var EASYRSA_REQ_OU "IT"' >> vars
echo 'set_var EASYRSA_REQ_EMAIL "dev@bigchaindb.com"' >> vars
```

3-4. Disable firewall   
`sudo ufw allow proto tcp from any to any port 80,443`

3-5. Generate dn, objectClass, ou for LDAP  
`sudo vim basedn.ldif`
```java
dn: ou=people,dc=selab,dc=cloud
objectClass: organizationalUnit
ou: people

dn: ou=groups,dc=selab,dc=cloud
objectClass: organizationalUnit
ou: groups
```

3-6. Apply the configuration  
`sudo ldapadd -x -D cn=admin,dc=selab,dc=cloud -W -f basedn.ldif`

3-7. Start the apache2  
`sudo service apache2 start`  

3-8. Check the phpldapadmin on http://localhost/phpldapadmin/
![img.png](../../../Assets/images/PHPLDAPADMIN.png)  

3-9. Create new ou(Organization Units) in phpldapadmin  
click to `Create new entry here` -> Select `Generic : Organization Unit`  

![img_1.png](../../../Assets/images/new-organization-units.png)  
Then fill up the form - Alias for new organization unit.  






---

## Remark


---

## Reference


---
