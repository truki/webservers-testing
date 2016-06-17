# Course's notes
---

## Apache 2.0

### Check Apache running??

check process:

```ps -fe | grep apache```

check service:

```service apache2 status```

here checks ports where Apache is listening

```netstat -putane | grep apache```

### Change ipv6 configuration to ipv4

``` cd /etc/apache2/
    # with this try to find "listen" string
    grep -i - R listen *
    vi ports.conf
    # change "Listen 80" to force ipv4
    Listen 0:0:0:0:80```

### Check syntax

```apachectl -t```

### Check version

``` dpkg -l | grep apache ```

o

``` apachectl -v ```

### Operating modules

Three kinds of operating modes. By default, before was prefork, but now there are two more: mpm_worker and mpm_event, and this last is the default module actually.

to disable an operating module:

``` a2dismod mpm_event ```

This generates an eror, because Apache need almost one operating module configured.

To re-enable

``` a2enmod mpm_event ```

It's not possible to execute two operating modules at the same time. Only one.

To see, which operating module is enabled you can:

``` ls -l /etc/apache2/mods-enabled | grep mpm ```
Operating modules types:

* **mpm_prefork** module: only this sure that php works good, because PHP don't work with threads. Memory problem. Every process respond to one request... we need too much RAM memory to give answer to every request.

  With **apache2-utils** we can monitor the performance of apache web server. It's usefull to run from other machine not from self.

  ```bash ab -c 10 -n 100 http://localhost:80 ```

  where:

  -c: Concurrency

  -n: # request



* **mpm_worker** module: use better performance of threads instead of forks processes.

* **mpm_event** module: better version of mpm_worker.

  Good article: [Apache vs Nginx] https://www.digitalocean.com/community/tutorials/apache-vs-nginx-practical-considerations


### More about modules

to see modules loaded:

```bash
root@vagrant-ubuntu-trusty-64:/etc/apache2/mods-available# apache2ctl -M
AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 10.0.2.15. Set the 'ServerName' directive globally to suppress this message
Loaded Modules:
 core_module (static)
 so_module (static)
 watchdog_module (static)
 http_module (static)
 log_config_module (static)
 logio_module (static)
 version_module (static)
 unixd_module (static)
 access_compat_module (shared)
 alias_module (shared)
 auth_basic_module (shared)
 authn_core_module (shared)
 authn_file_module (shared)
 authz_core_module (shared)
 ...
 ...
```

the result list indicate between parenthesis that the module could be compiled with apache code (**static**) or the module is loaded dinamically (**shared**).


### Others...

Show **OS information**
```
Servertokens OS
```

## Nginx

### Checks nginx is up and running

Check the service navigating to url

``` http://localhost ```

Checking:

``` service nginx status ```

### Check version

``` nginx -v (-V with more details) ```

or

``` apt-cache show nginx ```

### Recomendation

Add ppa official repositories to download the newer version.
```
sudo -s
nginx=stable # use nginx=development for latest development version
add-apt-repository ppa:nginx/$nginx
apt-get update
apt-get install nginx
```

### Configuration

Based on context {}, where is possible to inherit
Differents contexts, main (without curlybrackets), http, server, location...
All directive that define for example in http context

Possibles problems with directive:

* Directive used in error context.
* olders directives used in new releases.

# FastCGI

Evolution of CGI, is a protocol where you are making a proxy of differents request's types (PHP, ...)
Inconvenient --> overload

# PHP with Apache

we have two options if we choose FastCGI:
* PHP-FPM:
We need to modify provision Vagrant file with:

```
sudo add-apt-repository -y ppa:ondrej/apache2
sudo add-apt-repository -y ppa:ondrej/php5-5.6
sudo apt-get install -y apache2
sudo apt-get install -y php5-fpm \
     php5-mcrypt php5-mysql php5-gd php5-curl
```

after that

``` vagrant up ```

We need to create a root folder for this virtual host that wi will create

``` mkdir -p /var/www/ow-fastcgi.com/public```

Now create a virtualhost ow-fastcgi ne sites-available:

```
<VirtualHost *:80>
    ServerName ow-fastcgi.com
    ServerAlias www.ow-fastcgi.com


    DocumentRoot /var/www/ow-fastcgi.com/public

    <Directory /var/www/ow-fastcgi.com/public>
        Options -Indexes +FollowSymLinks +MultiViews
        AllowOverride All
        Require all granted
    </Directory>

    <FilesMatch \.php$>
        # 2.4.10+ can proxy to unix socket
        # SetHandler "proxy:unix:/var/run/php5-fpm.sock|fcgi://localhost/"

        # Else we can just use a tcp socket:
        SetHandler "proxy:fcgi://127.0.0.1:9000"
    </FilesMatch>

    ErrorLog ${APACHE_LOG_DIR}/example.com-error.log

    # Possible values include: debug, info, notice, warn, error, crit,
    # alert, emerg.
    LogLevel warn

    CustomLog ${APACHE_LOG_DIR}/ow-fastcgi.com-access.log combined

</VirtualHost>
```


* FCGI (mod_fcgid)
