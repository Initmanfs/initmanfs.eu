+++
author = "arthur"
categories = ["linux"]
date = "2017-09-24"
description = ""
draft = true
featured = ""
featuredalt = ""
featuredpath = "/img/posts/mailserver-on-ubuntu-16-04.en"
linktitle = ""
title = "Mailserver on Ubuntu 16.04"
type = "post"

+++

1. Setting up the host name

    ```shell
    echo mail.yourdomain.com > /etc/hostname
    ```

2. Adding our domain to /etc/hosts

    ```shell
    echo "127.0.0.1 localhost yourdomain.com  server.yourdomain.com" >> /etc/hosts
    ```

3. Setting up the mailname

    ```shell
    echo yourdomain.com > /etc/mailname
    ```

4. Update and install Software
Hit ‘No’ if asked to create a SSL certificate. Choose Internet Site and press ‘ok’ 2 times when asked by the postfix installer.

    ```shell
    apt-get update && apt-get upgrade && apt-get install postfix postfix-policyd-spf-perl postgrey dovecot-core dovecot-imapd opendkim opendkim-tools && postfix stop
    ```

5. Set up the DNS record
    * make A record with the ipv4
    * MX record fqdn 50
    * TXT for spf  "v=spf1 a mx ip4:1.2.3.4 -all"
    * DMARC with TXT _dmarc.yourdomain.com.
    ** "v=DMARC1; p=quarantine; rua=mailto:postmaster@yourdomain.com"

6. Create ssl cert with certbot or

    ```shell
    openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/mail.yourdomain.key -out /etc/ssl/certs/mail.yourdomain.pem
    ```

# Postfix configuration

Back up configuration

    ```shell
    cp /etc/postfix/master.cf /etc/postfix/master.cf_orig &&
    cp /etc/postfix/main.cf /etc/postfix/main.cf_orig
    vim /etc/postfix/main.cf
    ```  
paste

    ```
    #Base config
    myhostname = dystopia.arthurzinck.eu
    mydomain = arthurzinck.eu
    myorigin = /etc/mailname
    mydestination = $myhostname, $mydomain, localhost, localhost.$mydomain
    #relayhost =
    mynetworks = 127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128 79.137.36.55
    mailbox_size_limit = 0
    recipient_delimiter = +
    inet_interfaces = all
    relay_domains = $mydestination
    syslog_name=postfix/submission
    #Aliases / Recipients
    alias_maps = hash:/etc/aliases
    alias_database = hash:/etc/aliases
    local_recipient_maps = proxy:unix:passwd.byname $alias_maps
    #SSL/TLS
    smtpd_tls_cert_file=/etc/ssl/certs/dystopia.arthurzinck.eu.pem
    smtpd_tls_key_file=/etc/ssl/private/dystopia.arthurzinck.eu.key

    smtpd_use_tls=yes
    smtpd_tls_auth_only = yes
    smtpd_tls_session_cache_database = btree:${data_directory}/smtpd_scache
    smtp_tls_session_cache_database = btree:${data_directory}/smtp_scache
    smtpd_tls_security_level=may
    smtp_tls_security_level=may
    smtpd_tls_protocols = !SSLv2, !SSLv3, !TLSv1, !TLSv1.1
    smtpd_tls_wrappermode=no
    smtpd_sasl_type=dovecot
    smtpd_sasl_path=private/auth
    smtpd_sasl_auth_enable=yes
    milter_macro_daemon_name=ORIGINATING
    #Security and Anti-Spam cinfig
    policy-spf_time_limit = 3600s
    smtpd_helo_required = yes
    smtpd_tls_mandatory_ciphers = medium
    tls_medium_cipherlist = AES128+EECDH:AES128+EDH
    smtpd_tls_mandatory_protocols = !SSLv2,!SSLv3,!TLSv1,!TLSv1.1
    smtpd_recipient_restrictions =
     reject_non_fqdn_recipient
    # reject_unknown_recipient_domain
     permit_mynetworks
     permit_sasl_authenticated
    # reject_unauth_destination
    # check_policy_service unix:private/policy-spf
    # check_policy_service inet:127.0.0.1:10023

    #smtpd_helo_restrictions =
    # permit_mynetworks
    # reject_non_fqdn_helo_hostname
    # reject_invalid_helo_hostname

    #smtpd_client_restrictions=
    # permit_mynetworks
    # permit_sasl_authenticated
    # reject_unknown_client_hostname

    #smtpd_data_restrictions =
    # reject_unauth_pipelining

    #DKIM
    milter_default_action = accept
    milter_protocol = 2
    smtpd_milters = inet:localhost:8891
    non_smtpd_milters = inet:localhost:8891
    ```
    ```shell
    vim /etc/postfix/master,cf
    ```  

this line must be active:

    ```
    smtp inet n - - - - smtpd
    ```  

We uncomment the following:

    ```
    submission inet n - - - - smtpd
    ```  

and add those 2 lines at the end of the file (there are 2 whitespaces
before the second line).

    ```
    policy-spf unix - n n - - spawn
      user=nobody argv=/usr/sbin/postfix-policyd-spf-perl
    ```
[Postfix configuration help][postfixConf]

# Dovecot configuration

```shell
cp /etc/dovecot/dovecot.conf /etc/dovecot/dovecot.conf_orig
```

```shell
vim /etc/dovecot/dovecot.conf
```
and paste

```
disable_plaintext_auth = no
mail_privileged_group = mail
mail_location = mbox:~/mail:INBOX=/var/mail/%u
userdb {
driver = passwd
}

passdb {
args = %s
driver = pam
}
protocols = " imap"

protocol imap {
mail_plugins = " autocreate"
}
plugin {
autocreate = Trash
autocreate2 = Sent
autosubscribe = Trash
autosubscribe2 = Sent
}

service auth {
unix_listener /var/spool/postfix/private/auth {
group = postfix
mode = 0660
user = postfix
}
}
ssl=required
ssl_cert = </etc/ssl/certs/mail.yourdomain.pem
ssl_key = </etc/ssl/private/mail.yourdomain.key
```

### Restart Dovecot

```shell
service dovecot restart
```

# Add Mail user

```shell
adduser --gecos '' --shell /bin/false yourname
```

# set up aliases

```shell
vim /etc/aliases
```
paste

```
mailer-daemon: postmaster
postmaster: root
nobody: root
hostmaster: root
usenet: root
news: root
webmaster: root
www: root
ftp: root
abuse: root
noc: root
security: root
admin: root
root: yourname
```
run :

```shell
newaliases
```
# Creating the directory and files

```shell
mkdir /etc/opendkim

vim /etc/opendkim/KeyTable
```

We enter the following line (all in the first line):

```
default._domainkey.mail.yourdomain.com mail.yourdomain.com:default:/etc/opendkim/default.private
```
```shell
vim /etc/opendkim/SigningTable
```
We enter the following line:
```
*@yourdomain.com default._domainkey.mail.yourdomain.com
```

# Generating the key pair

```shell
opendkim-genkey -s default -d mail.yourdomain.com -D /etc/opendkim
```

Changing ownership of the private key file

```shell
chown opendkim:opendkim /etc/opendkim/default.private
```

Configuration in opendkim

```shell
vim /etc/default/opendkim
```

We add the line below to the configuration

```
SOCKET="inet:8891@localhost"
```

Configuration in opendkim.conf

```shell
vim /etc/opendkim.conf
```

We fill the file with our configuration

```
Syslog yes
SyslogSuccess Yes
LogWhy yes

UMask 002

KeyTable refile:/etc/opendkim/KeyTable
SigningTable refile:/etc/opendkim/SigningTable
Selector default
X-Header no
SignatureAlgorithm rsa-sha256

Canonicalization relaxed/simple
Mode sv
AutoRestart Yes
AutoRestartRate 5/1h
InternalHosts 127.0.0.1, localhost, yourdomain.com, mail.yourdomain.com

OversignHeaders From
```

Setting the DKIM DNS record

```shell
cat /etc/opendkim/default.txt
```

This is our public key which will be used to verify the signature in our emails.

```dns
default._domainkey IN TXT "v=DKIM1; k=rsa;
p=MIGfHL0GCSqGSIb3DQESYJFOA4GNADCBiQKBgQDS+vPyWRs7w32xomf2oZIexmS2TuQAXKPiQ3AXn4j25NOReXdgKxIqAwl3O7dQtgluWw+TH85Mrbmx5UgwaaLenj9cfe2IRvx7hvkj7+6i0XQqrWqZlMw+QAJxAGhfa/GVTYa+/7PFWfXLoqoBW5arE+wO20O2uw5Ik62HjkKZbQIDAQAB" ; ----- DKIM key default for mail.yourdomain.com
```

Now we add a new TXT record:

As ‘name’ we put (there is a ‘dot’ at the end):
```
default._domainkey.mail.yourdomain.com.
```
As ‘text’ we add (use your public key from /etc/opendkim/default.txt):
```
"v=DKIM1; k=rsa; p=MIGfHL0GCSqGSIb3DQESYJFOA4GNADCBiQKBgQDS+vPyWRs7w32xomf2oZIexmS2TuQAXKPiQ3AXn4j25NOReXdgKxIqAwl3O7dQtgluWw+TH85Mrbmx5UgwaaLenj9cfe2IRvx7hvkj7+6i0XQqrWqZlMw+QAJxAGhfa/GVTYa+/7PFWfXLoqoBW5arE+wO20O2uw5Ik62HjkKZbQIDAQAB"
```
Starting

```shell
postfix start && service opendkim restart
```
then to connect it yourname password of unix account

![][image1]

[postfixConf]: http://www.postfix.org/postconf.5.html "Postfix Configuration Documentation"

[image1]: /img/posts/mailserver-on-ubuntu-16-04/2017-08-09-183744_944x417_scrot.png
