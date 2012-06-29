we would like to start-up an Apache server. Deploy a simple website that restrict access to several resources pointed inside the webpage. We want to define the AuthN/AuthZ procedure with access to LDAP server, and the users to be checked there certificate that we issue from our Certificate Authority.

Steps:

1. We create our own CA. We follow the steps in this very old, but still valid and simple, guides:
 *  [Creating and using a self signed SSL Certificates in Debian](http://www.debian-administration.org/articles/284)
 *  [CA with Openssl](http://www.debian-administration.org/articles/618)


So we edit the file /usr/lib/ssl/misc/CA.pl adding this lines:

    --bash  
     $DAYS="-days 730";     # 2 year  
     $CADAYS="-days 3650";  # 10 years  
     $CATOP="/etc/ssl/ca";  

We edit /etc/ssl/openssl.cnf:  

    --bash     
    dir = /etc/ssl/ca  
    default_days = 730  


Now, LDAP server setup  
To use SASL correctly, you need to put the certificates generated above in the correct places:

    --bash  
    mv /etc/ssl/newcert.pem /etc/ldap/servercrt.pem  
    mv /etc/ssl/newreq.pem /etc/ldap/serverkey.pem  
    cp /etc/ssl/ca/cacert.pem /etc/ldap/cacert.pem  
    chmod go-r /etc/ldap/serverkey.pem  
    chown openldap /etc/ldap/serverkey.pem  
    chmod a+r /etc/ldap/servercrt.pem  


Edit /etc/default/slapd to include these lines:  

    --bash  
    SLAPD_SERVICES="ldaps:///"  
    SLAPD_OPTIONS="4"  

Edit /usr/share/slapd/slapd.conf to include these lines:  

    --bash  
    TLSCACertificateFile    /etc/ldap/cacert.pem  
    TLSCertificateFile      /etc/ldap/servercrt.pem  
    TLSCertificateKeyFile   /etc/ldap/serverkey.pem  

2. Lets write a very simple website that access to resources. 
 * Install apache and php5 module.

    --bash
    sudo apt-get install apache2
    sudo apt-get install apache2-mpm-prefork 
    sudo apt-get install libapache2-mod-php5

