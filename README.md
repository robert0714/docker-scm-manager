# Notation

##　SELinux's Effect

於系統有SELinux需要以下指令讓執行nginx時可以讀取後來增加的檔案與資料夾:
(https://serverfault.com/questions/540537/nginx-permission-denied-to-certificate-files-for-ssl-configuration )

```bash

$ restorecon -v -R /etc/nginx/conf/certs

worker_processes auto;

http {

    ...

    server {
        listen              443 ssl;
        keepalive_timeout   70;

        ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers         AES128-SHA:AES256-SHA:RC4-SHA:DES-CBC3-SHA:RC4-MD5;
        ssl_certificate     /usr/local/nginx/conf/cert.pem;
        ssl_certificate_key /usr/local/nginx/conf/cert.key;
        ssl_session_cache   shared:SSL:10m;
        ssl_session_timeout 10m;

        ...
    }
    
```
error log show the below：

```
nginx connect() to 127.0.0.1:8180 failed (13: Permission denied) while connecting to upstream.

```
This should solve the problem:
setsebool -P httpd_can_network_connect 1
Details
I checked for errors in the SELinux logs:

```
sudo cat /var/log/audit/audit.log | grep nginx | grep denied

```
And found that running the following commands fixed my issue:

```
sudo cat /var/log/audit/audit.log | grep nginx | grep denied | audit2allow -M mynginx sudo semodule -i mynginx.pp

```
References:

1. http://blog.frag-gustav.de/2013/07/21/nginx-selinux-me-mad/
1. https://wiki.gentoo.org/wiki/SELinux/Tutorials/Where_to_find_SELinux_permission_denial_details
1. http://wiki.gentoo.org/wiki/SELinux/Tutorials/Managing_network_port_labels
1. http://www.linuxproblems.org/wiki/Selinux

I’ve run into this problem too. Another solution is to toggle the SELinux boolean value for httpd network connect to on (Nginx uses the httpd label).

```
setsebool httpd_can_network_connect on

```
To make the change persist use the -P flag.


```
setsebool httpd_can_network_connect on -P

```
You can see a list of all available SELinux booleans for httpd using

```
getsebool -a | grep httpd

```
