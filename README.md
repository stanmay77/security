# Домашнее задание к занятию «Элементы безопасности информационных систем»

1. Установите плагин Bitwarden для браузера. Зарегестрируйтесь и сохраните несколько паролей.

![](bitwarden.png)


2. Установите Google Authenticator на мобильный телефон. Настройте вход в Bitwarden-акаунт через Google Authenticator OTP.

![](gauth.png)

3. Установите apache2, сгенерируйте самоподписанный сертификат, настройте тестовый сайт для работы по HTTPS.

```
$ sudo systemctl status apache2
* apache2.service - The Apache HTTP Server
   Loaded: loaded (/lib/systemd/system/apache2.service; enabled; vendor preset: enabled)
  Drop-In: /lib/systemd/system/apache2.service.d
           `-apache2-systemd.conf
   Active: active (running) since Sat 2023-03-11 05:16:36 PST; 28s ago
 Main PID: 9539 (apache2)
    Tasks: 55 (limit: 4915)
   CGroup: /system.slice/apache2.service
           |-9539 /usr/sbin/apache2 -k start
           |-9540 /usr/sbin/apache2 -k start
           `-9541 /usr/sbin/apache2 -k start

Mar 11 05:16:36 op-net.com systemd[1]: Starting The Apache HTTP Server...
Mar 11 05:16:36 op-net.com systemd[1]: Started The Apache HTTP Server.

$ sudo a2enmod ssl
Considering dependency setenvif for ssl:
Module setenvif already enabled
Considering dependency mime for ssl:
Module mime already enabled
Considering dependency socache_shmcb for ssl:
Enabling module socache_shmcb.
Enabling module ssl.
See /usr/share/doc/apache2/README.Debian.gz on how to configure SSL and create self-signed certificates.
To activate the new configuration, you need to run:
  systemctl restart apache2
$ sudo systemctl restart apache2
$ sudo openssl req -x509 -nodes -days 365 \
> -newkey rsa:2048 -keyout /etc/ssl/private/apache-selfsigned.key \
> ^C                                       
$ sudo openssl req -x509 -nodes -days 356 -newkey rsa:2048 -keyout /etc/ssl/private/apache-selfsigned.key -out /etc/ssl/certs/apache-selfsigned.crt -subj "/C=US/ST=NYC/L=NYC/O=CyberLand/OU=Org/CN=nosite.com"
Can't load /home/stan/.rnd into RNG
140201264374208:error:2406F079:random number generator:RAND_load_file:Cannot open file:../crypto/rand/randfile.c:88:Filename=/home/stan/.rnd
Generating a RSA private key
.................................................................................................................+++++
.....................................................................................................................+++++
writing new private key to '/etc/ssl/private/apache-selfsigned.key'
-----
$ sudo nano /etc/apache2/sites-available/default-ssl.conf
```

Конфиг apache для ssl подключения:

```
<IfModule mod_ssl.c>
        <VirtualHost _default_:443>
                ServerAdmin webmaster@localhost

                DocumentRoot /var/www/html
                ErrorLog ${APACHE_LOG_DIR}/error.log
                CustomLog ${APACHE_LOG_DIR}/access.log combined
                SSLEngine on

                SSLCertificateFile      /etc/ssl/certs/apache-selfsigned.crt
                SSLCertificateKeyFile /etc/ssl/private/apache-selfsigned.key
                <FilesMatch "\.(cgi|shtml|phtml|php)$">
                                SSLOptions +StdEnvVars
                </FilesMatch>
                <Directory /usr/lib/cgi-bin>
                                SSLOptions +StdEnvVars
                </Directory>

                </VirtualHost>
</IfModule>
```

Далее включаем ssl конфиг:
```
$ sudo a2ensite default-ssl.conf
Enabling site default-ssl.
To activate the new configuration, you need to run:
  systemctl reload apache2
$ systemctl reload apache2
```

4. Проверьте на TLS-уязвимости произвольный сайт в интернете (кроме сайтов МВД, ФСБ, МинОбр, НацБанк, РосКосмос, РосАтом, РосНАНО и любых госкомпаний, объектов КИИ, ВПК и т. п.).

```
$ ./testssl.sh/testssl.sh -U --sneaky https://google.com

###########################################################
    testssl.sh       3.2rc2 from https://testssl.sh/dev/

      This program is free software. Distribution and
             modification under GPLv2 permitted.
      USAGE w/o ANY WARRANTY. USE IT AT YOUR OWN RISK!

       Please file bugs @ https://testssl.sh/bugs/

###########################################################

 Using "OpenSSL 1.0.2-bad (1.0.2k-dev)" [~183 ciphers]
 on nyc-436944:./testssl.sh/bin/openssl.Linux.x86_64
 (built: "Sep  1 14:03:44 2022", platform: "linux-x86_64")


Testing all IPv4 addresses (port 443): 172.217.203.101 172.217.203.113 172.217.203.139 172.217.203.102 172.217.203.138 172.217.203.100
---------------------------------------------------------------------
 Start 2023-03-11 05:47:31        -->> 172.217.203.101:443 (google.com) <<--

 Further IP addresses:   172.217.203.100 172.217.203.138 172.217.203.102 172.217.203.139
                         172.217.203.113 2607:f8b0:400c:c07::66 2607:f8b0:400c:c07::64
                         2607:f8b0:400c:c07::71 2607:f8b0:400c:c07::8a 
 rDNS (172.217.203.101): uf-in-f101.1e100.net.
 Service detected:       HTTP


 Testing vulnerabilities 

 Heartbleed (CVE-2014-0160)                not vulnerable (OK), no heartbeat extension
 CCS (CVE-2014-0224)                       not vulnerable (OK)
 Ticketbleed (CVE-2016-9244), experiment.  not vulnerable (OK)
 ROBOT                                     not vulnerable (OK)
 Secure Renegotiation (RFC 5746)           supported (OK)
 Secure Client-Initiated Renegotiation     not vulnerable (OK)
 CRIME, TLS (CVE-2012-4929)                not vulnerable (OK)
 BREACH (CVE-2013-3587)                    no gzip/deflate/compress/br HTTP compression (OK)  - only supplied "/" tested
 POODLE, SSL (CVE-2014-3566)               not vulnerable (OK)
 TLS_FALLBACK_SCSV (RFC 7507)              Downgrade attack prevention supported (OK)
 SWEET32 (CVE-2016-2183, CVE-2016-6329)    VULNERABLE, uses 64 bit block ciphers
 FREAK (CVE-2015-0204)                     not vulnerable (OK)
 DROWN (CVE-2016-0800, CVE-2016-0703)      not vulnerable on this host and port (OK)
                                           make sure you don't use this certificate elsewhere with SSLv2 enabled services, see
                                           https://search.censys.io/search?resource=hosts&virtual_hosts=INCLUDE&q=B61F1C6F6BAD0CA9D74E3115C8934D994C993B6F265226E3F0311645F1D44F83
 LOGJAM (CVE-2015-4000), experimental      not vulnerable (OK): no DH EXPORT ciphers, no DH key detected with <= TLS 1.2
 BEAST (CVE-2011-3389)                     TLS1: ECDHE-ECDSA-AES128-SHA ECDHE-ECDSA-AES256-SHA
                                                 ECDHE-RSA-AES128-SHA ECDHE-RSA-AES256-SHA AES128-SHA
                                                 AES256-SHA DES-CBC3-SHA 
                                           VULNERABLE -- but also supports higher protocols  TLSv1.1 TLSv1.2 (likely mitigated)
 LUCKY13 (CVE-2013-0169), experimental     potentially VULNERABLE, uses cipher block chaining (CBC) ciphers with TLS. Check patches
 Winshock (CVE-2014-6321), experimental    not vulnerable (OK)
 RC4 (CVE-2013-2566, CVE-2015-2808)        no RC4 ciphers detected (OK)


 Done 2023-03-11 05:48:08 [  40s] -->> 172.217.203.101:443 (google.com) <<--

```

5. Установите на Ubuntu SSH-сервер, сгенерируйте новый приватный ключ. Скопируйте свой публичный ключ на другой сервер. Подключитесь к серверу по SSH-ключу.

```
$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/stan/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/stan/.ssh/id_rsa.
Your public key has been saved in /home/stan/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:PIk+o1Vj1T9oCWRdz92SvvodEMN/c43bNo1aJaH1RdA stan@nyc-436944.op-net.com
The key's randomart image is:
+---[RSA 2048]----+
|          o. .oo.|
|         o .o  =E|
|          o .+= *|
|       o o . B+=o|
|      . S   =.*o*|
|     . o o .  .O=|
|      =       =o+|
|     o o     + oo|
|    .       o.. .|
+----[SHA256]-----+

$ ssh-copy-id stan@XXX.XXX.XXX.XXX
```

6. Переименуйте файлы ключей из задания 5. Настройте файл конфигурации SSH-клиента так, чтобы вход на удалённый сервер осуществлялся по имени сервера.

```
$ mv /home/stan/.ssh/id_rsa /home/stan/.ssh/myubuntu_rsa
$ touch ~/.ssh/config && chmod 600 ~/.ssh/config
$ nano .ssh/config

Host myubuntu
     HostName XXX.XXX.XXX.XXX
     User stan
     IdentityFile ~/.ssh/myubuntu_rsa
     
$ ssh myubuntu
```

7. Соберите дамп трафика утилитой tcpdump в формате pcap, 100 пакетов. Откройте файл pcap в Wireshark.

```
$ sudo tcpdump -c 100 -w dump1.pcap -i primary
[sudo] password for stan: 
tcpdump: listening on primary, link-type EN10MB (Ethernet), capture size 262144 bytes
100 packets captured
107 packets received by filter
0 packets dropped by kernel
```




