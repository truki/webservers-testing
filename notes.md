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

Three kinds of operating modes. By default, before was prefork, but now there are two more mpm_worker and mpm_event.

to disable an operating module:

``` a2dismod mpm_event ```

This generates an eror, because Apache need almost one operating module configured.

To re-enable

``` a2enmod mpm_event ```

It's not possible to execute two operating modules at the same time. Only one.

To see, which operating module is enabled you can:

´´´ ls -l /etc/apache2/mods-enabled | grep mpm ```
Operating modules types:

* **mpm_prefork** module: only this sure that php works good, because PHP don't work with threads. Memory problem. Every process respond to one request... we need too much RAM memory to give answer to every request.

  With **apache2utils** we can monitor the performance of apache web server. It's usefull to run from other machine not from self.

  ``` ab -c 10 -n 100 http://localhost:80 ```

  where:

  -c: Concurrency

  -n: # request



* **mpm_worker** module: use better performance of threads instead of forks processes.

* **mpm_event** module: better version of mpm_worker.

  Good article: [Apache vs Nginx] https://www.digitalocean.com/community/tutorials/apache-vs-nginx-practical-considerations
