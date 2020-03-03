# Setting up a Filter for X-Forwarded-For ip infail2ban

The specific filter is very simple and will match the second IP in the access logs:

``` bash
# filter.d/wordpress.conf
[Definition]
failregex = \S+ <HOST> .* "POST .*wp-login.php
ignoreregex =
```

``` bash
# jail.d/wordpress.conf
[wordpress]
enabled = true
port = http,https
filter = wordpress
action = iptables-multiport[name=wordpress, port="http,https", protocol=tcp]
logpath = /home/*/logs/*access_log
maxretry = 3
findtime = 600
action = %(action_)s
         %(action_abuseipdb)s[abuseipdb_apikey="ABUSEIPDB_API_KEY", abuseipdb_category="18,21",comment="WordPress brute-force."]
```

Main article and filter solution source - [Source](https://blog.rimuhosting.com/2016/11/02/using-fail2ban-on-wordpress-wp-login-php-and-xmlrpc-php/)

They key target for splitting the IP's is \S+ - [Source](https://fail2ban.readthedocs.io/en/latest/filters.html)

Example log which will match the X-Forwarded-For IP:

``` log
172.69.34.204 125.161.130.94 - [24/Feb/2020:09:20:54 +0000] "POST /wp-login.php HTTP/1.1" 200 Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:62.0) Gecko/20100101 Firefox/62.0 4354 PHP/7.3.15 582692
162.158.166.215 111.88.125.203 - [24/Feb/2020:10:23:13 +0000] "POST /wp-login.php HTTP/1.1" 200 Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:62.0) Gecko/20100101 Firefox/62.0 4354 PHP/7.3.15 1283893
```

## Test scenarios:

``` bash
fail2ban-regex /home/m0sh1x2.com/logs/m0sh1x2.com.access_log /etc/fail2ban/filter.d/wordpress.con
```

Output:

``` bash
Running tests
=============

Use   failregex filter file : wordpress, basedir: /etc/fail2ban
Use         log file : /home/m0sh1x2.com/logs/m0sh1x2.com.access_log
Use         encoding : UTF-8


Results
=======

Failregex: 472 total
|-  #) [# of hits] regular expression
|   1) [472] \S+ <HOST> .* "POST .*wp-login.php
`-

Ignoreregex: 0 total

Date template hits:
|- [# of hits] date format
|  [5759] Day(?P<_sep>[-/])MON(?P=_sep)ExYear[ :]?24hour:Minute:Second(?:\.Microseconds)?(?: Zone offset)?
`-

Lines: 5759 lines, 0 ignored, 472 matched, 5287 missed
[processed in 1.80 sec]

Missed line(s): too many to print.  Use --print-all-missed to print all 5287 lines
```