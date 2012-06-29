we would like to use Heimdal Kerberos on Debian for SSH. 

We need 3 machines:

1.  master (lxid01.devops.gsi.de): runs the services kadmin and kdc.
2.  sshserver (lxb001.devops.gsi.de): SSH server
3.  sshclient (lxb003.devops.gsi.de): SSH client

Requirements for all hosts:

+ Exact time. As minimum run ntpdate at startup, for example: 

    --bash
    ntpdate -u 130.149.17.8

+ DNS configuration: forward and reversal lookup is needed for all machines.

    --bash
    root@lxid01:/home/devops# ifconfig | grep 10.1.1
    inet addr:10.1.1.19  Bcast:10.1.1.255  Mask:255.255.255.0
    root@lxid01:/home/devops# host 10.1.1.19
    19.1.1.10.in-addr.arpa domain name pointer lxid01.devops.gsi.de.
    root@lxid01:/home/devops# host lxid01.devops.gsi.de
    lxid01.devops.gsi.de has address 10.1.1.19

+ Hostname configuration: /etc/hostname and /etc/hosts has to match the DNS name.

Now we get the recipe _heimdal_ from gitorius.

Configuration of the machines: 

    --bash
    devops@lxcm01:~$ knife status -r 
    24 minutes ago, lxb003.devops.gsi.de, lxb003.devops.gsi.de, 10.1.1.6, , recipe[heimdal::client]., debian 6.0.2.
    22 minutes ago, lxcm01.devops.gsi.de, lxcm01.devops.gsi.de, 10.1.1.3, , ., debian 6.0.4.
    8 minutes ago, lxid01.devops.gsi.de, lxid01.devops.gsi.de, 10.1.1.19, , recipe[heimdal::server], recipe[ntp]., debian 6.0.4.
    5 minutes ago, lxb001.devops.gsi.de, lxb001.devops.gsi.de, 10.1.1.4, , recipe[heimdal::client]., debian 6.0.4.
    1 minute  ago, lxdns01.devops.gsi.de, lxdns01.devops.gsi.de, 10.1.1.2, , recipe[apt], recipe[lazydns], recipe[resolv], recipe[ntp]., debian 6.0.4


We have to edit the attributes according to our configuration. We could do it in several ways: defining roles, by json configuration files. We will edit the nodes accordingly. 

The server:

    --bash 
    devops@lxcm01:~$ knife node edit lxid01.devops.gsi.de

    {
      "normal": {
        "tags": [
    
        ],
        "krb5": {
          "adm_server": "lxid01.devops.gsi.de",
          "realm_name": "DEVOPS.GSI.DE",
          "kdc": "lxid01.devops.gsi.de"
        }
          },
      "name": "lxid01.devops.gsi.de",
      "chef_environment": "_default",
      "run_list": [
        "recipe[heimdal::server]",
        "recipe[ntp]"
      ]
    }


The clients (lxb001.devops.gsi.de, lxb003.devops.gsi.de)

    --bash 
    devops@lxcm01:~$ knife node edit lxid01.devops.gsi.de

    {
      "normal": {
        "tags": [

        ],
        "krb5": {
          "adm_server": "lxid01.devops.gsi.de",
          "realm_name": "DEVOPS.GSI.DE",
          "kdc": "lxid01.devops.gsi.de"
        }
      },
      "name": "lxb001.devops.gsi.de",
      "chef_environment": "_default",
      "run_list": [
        "recipe[heimdal::client]"
      ]
    }


Inside the recipe the key for the database where all the principals and passwords will be saved is created, and saved in: /etc/heimdal-kdc/stash, with a random key.

    --bash 
    root@lxid01:/home/devops# file /etc/heimdal-kdc/stash
    /etc/heimdal-kdc/stash: data

We can see all the kerberos processes running: 

    --bash
    root@lxid01:/home/devops# ps -aux | grep heimdal
    root     11123  0.0  0.5  63084  3016 ?        S    17:09   0:00 /usr/lib/heimdal-servers/kdc --config-file=/etc/heimdal-kdc/kdc.conf
    root     11124  0.0  0.5  60924  2872 ?        S    17:09   0:00 /usr/lib/heimdal-servers/kpasswdd
    root     11184  0.0  0.1   7548   836 pts/1    S+   17:29   0:00 grep heimdal
    root     24714  0.0  0.5  65272  2996 ?        S    Apr04   0:00 /usr/lib/heimdal-servers/kadmind

The process kadmind, also called kerberos-adm is started via inetd. Debians openbsd-inetd is already preconfigured.

    --bash 
    root@lxid01:/home/devops# lsof -i :749
    COMMAND   PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
    inetd   11154 root    7u  IPv4 239333      0t0  TCP *:kerberos-adm (LISTEN)
    kadmind 24714 root    3u  IPv6  73670      0t0  TCP *:kerberos-adm (LISTEN)


There are two important config files: /etc/default/heimdal-kdc and /etc/heimdal-kdc/kdc.conf. 

    --bash
    root@lxid01:/home/devops# cat /etc/heimdal-kdc/kdc.conf 
    [kdc]
    	database = {
	          	dbname = /var/lib/heimdal-kdc/devops.gsi.de
	    	realm =  DEVOPS.GSI.DE
	    	acl_file = /etc/heimdal-kdc/kadm5.acl
    		key_stash_file = /etc/heimdal-kdc/stash
     		max_life = 24h 0m 0s
    		max_renewable_life = 7d 0h 0m 0s
		    master_key_type = des3-cbc-sha1
	    	supported_enctypes = des3-hmac-sha1:normal des-cbc-crc:normal des3-cbc-sha1:normal aes256-cts-hmac-sha1-96 
           default_principal_flags = +preauth

    }

    [logging]
    	kdc		= FILE:/var/log/heimdal/kdc.log
    	kadmind		= FILE:/var/log/heimdal/kadmin.log
    	kpasswdd	= FILE:/var/log/heimdal/kpasswdd.log
    	default		= FILE:/var/log/heimdal/heimdal.log


The basic configuration is stored in /etc/krb5.conf. This file has to be the same on all three machines. This file is created during the execution of the recipe. Since we have set the attributes correctly we should have something similar to this:  

    --bash
    root@lxid01:/home/devops# cat /etc/krb5.conf 
    [libdefaults]
    	default_realm		= DEVOPS.GSI.DE
            ticket_lifetime		= 28800
    	dns_lookup_realm 	= false
    	dns_lookup_kdc 		= false
    	fcc-mit-ticketflags 	= true

    m[realms]
    	DEVOPS.GSI.DE = {
    	 	kdc 		     = lxid01.devops.gsi.de
    		admin_server 	     = lxid01.devops.gsi.de
    		default_domain 	     = devops.gsi.de
    	}

    [domain_realm]
    	.devops.gsi.de = DEVOPS.GSI.DE
    	devops.gsi.de = DEVOPS.GSI.DE

    [logging]


    default = FILE:/var/log/heimdal/default.log

We will use the _kadmin_ tool for configuration. We initialize the database:

    --bash
    root@lxid01:/home/devops# kadmin -l
    kadmin> list *
    foo
    default
    amontiel
    kadmin/admin
    kadmin/hprop
    amontiel/admin
    kadmin/changepw
    changepw/kerberos
    WELLKNOWN/ANONYMOUS
    krbtgt/DEVOPS.GSI.DE
    default@~[--realm-max-renewable-life=10m
    host/lxb001.devops.gsi.de
    kadmin/admin@~[--realm-max-renewable-life=10m
    kadmin/hprop@~[--realm-max-renewable-life=10m
    kadmin/changepw@~[--realm-max-renewable-life=10m
    changepw/kerberos@~[--realm-max-renewable-life=10m
    WELLKNOWN/ANONYMOUS@~[--realm-max-renewable-life=10m
    krbtgt/~[--realm-max-renewable-life=10m@~[--realm-max-renewable-life=10m
    

We change the password of the admin user with 'cpw kadmin/admin'. Then, we create a test user with 'add foo'.

    --bash 
    kadmin> list -l foo
            Principal: foo@DEVOPS.GSI.DE
    Principal expires: never
     Password expires: never
 Last password change: 2012-04-11 13:35:36 UTC
      Max ticket life: 1 day
   Max renewable life: 1 week
                 Kvno: 1
                Mkvno: unknown
Last successful login: never
    Last failed login: never
   Failed login count: 0
        Last modified: 2012-04-11 13:35:36 UTC
             Modifier: amontiel/admin@DEVOPS.GSI.DE
           Attributes: 
             Keytypes: aes256-cts-hmac-sha1-96(pw-salt), des3-cbc-sha1(pw-salt), arcfour-hmac-md5(pw-salt)
          PK-INIT ACL: 
              Aliases: 


In the SSH server: 

we have installed a heimdal client, through the recipe indicated above. The configuration file _/etc/krb5.cnf_ is already set.We are able to login with kadmin user from this machine, because we changed before the password in the Heimdal server. 
We login with kadmin/admin and create a principal inside kerberos for the host, the ssh server:


    --bash
    root@lxb001:/home/devops# kadmin -p kadmin/admin
    kadmin> list *
    kadmin/admin@DEVOPS.GSI.DE's Password: 
    foo
    default
    amontiel
    kadmin/admin
    kadmin/hprop
    amontiel/admin
    kadmin/changepw
    changepw/kerberos
    WELLKNOWN/ANONYMOUS
    krbtgt/DEVOPS.GSI.DE
    default@~[--realm-max-renewable-life=10m
    host/lxb001.devops.gsi.de
    kadmin/admin@~[--realm-max-renewable-life=10m
    kadmin/hprop@~[--realm-max-renewable-life=10m
    kadmin/changepw@~[--realm-max-renewable-life=10m
    changepw/kerberos@~[--realm-max-renewable-life=10m
    WELLKNOWN/ANONYMOUS@~[--realm-max-renewable-life=10m
    krbtgt/~[--realm-max-renewable-life=10m@~[--realm-max-renewable-life=10m
    kadmin> add --random-key host/lxb001.devops.gsi.de


The following configuration line is enough to activate Kerberos authN in SSHd: 

_echo "GSSAPIAuthentication yes" >> /etc/ssh/sshd_config_


