___
| [[Splunk]] | [[DNS - Open DNS - Mac Linux]] 
___
| [[Splunk and  SplunkForwarder]] 
# Squid's book at O'Reilly - 

## Quotes & Tips

> [!quote] 
> You definitely don’t want your Squid cache to be used as a spam relay. If it is, your IP address is likely to end up on one of the many mail-relay blacklists (MAPS, ORDB, spamhaus, etc.). In addition to email, there are a number of other TCP/IP services that Squid shouldn’t normally communicate with. These include IRC, Telnet, DNS, POP, and NNTP. Your policy regarding port numbers should be either to deny the known-to-be-dangerous ports and allow the rest, or to allow the known-to-be-safe ports and deny the rest.

### How Squid Matches Access Control Elements
> [!quote]
> Recall that Squid uses `OR` logic when searching ACL elements. Any single value in an acl can cause a match.
> 
> It’s the opposite for access rules, however. For http_access and the other rule sets, Squid uses `AND` logic.



___
### Introduction to squid access control elements
![[Pasted image 20241130012733.png]]

![[Pasted image 20241130012804.png]]

![[Pasted image 20241130012836.png]]

![[Pasted image 20241130012906.png]]

![[Pasted image 20241130012921.png]]

![[Pasted image 20241130013006.png]]
____





### - 1 Access control Rule Syntax


### - 2 The `acl All src 0/0`

```bash
acl All src 0/0
```

> [!quote] The `src 0/0 ACL` is an easy way to match each and every type of request.
![[Pasted image 20241130014356.png]]


### - 2.2 Access List Style

> [!quote]
> Squid’s access control syntax is very powerful. In most cases, you can probably think of two or more ways to accomplish the same thing. In general, you should put the more specific and restrictive access controls first. 
- Check the hint below
```shell
acl All src 0/0
acl Net1 src 1.2.3.0/24
acl Net2 src 1.2.4.0/24
acl Net3 src 1.2.5.0/24
acl Net4 src 1.2.6.0/24
acl WorkingHours time 08:00-17:00

http_access allow Net1 WorkingHours
http_access allow Net2 WorkingHours
http_access allow Net3 WorkingHours
http_access allow Net4
http_access deny All
```

> [!light] you might find it easier to maintain and understand the access control configuration if you write it like this: (  `!` is not sign)
```shell
http_access allow Net4
http_access deny !WorkingHours
http_access allow Net1
http_access allow Net2
http_access allow Net3
http_access deny All
```

> [!info] Using negation `!`
> Whenever you have a rule with two or more ACL elements, it’s always a good idea to follow it up with an opposite, more general rule. For example, the default Squid configuration denies cache manager requests that don’t come from the localhost IP address. You might be tempted to write it like this:
___
>```shell
>acl CacheManager proto cache_object
>acl Localhost src 127.0.0.1
>http_access deny CacheManager !Localhost
>```

> [!important] 
> However, the problem here is that you haven’t yet allowed the cache manager requests that do come from localhost. Subsequent rules may cause the request to be denied anyway. These rules have this undesirable behavior:

 ```shell
acl CacheManager proto cache_object
acl Localhost src 127.0.0.1
acl MyNet 10.0.0.0/24
acl All src 0/0
http_access deny CacheManager !Localhost
http_access allow MyNet
http_access deny All
```

> [!light]
> Since a request from localhost doesn’t match MyNet, it gets denied. A better way to write the rules is like this:
```shell
http_access allow CacheManager localhost
http_access deny CacheManager
http_access allow MyNet
http_access deny All
```


### - 3 Access Rule Syntax tips
![[Pasted image 20241130013410.png]]

![[Pasted image 20241130013448.png]]



____

### Slow & Fast Rule Checks
#### Faq
> [!faq]
> Internally, Squid considers some access rule checks fast, and others slow. The difference is whether or not Squid postpones its decision to wait for additional information. In other words, a slow check may be deferred while Squid asks for additional data, such as:
> 
> A reverse DNS lookup: the hostname for a client’s IP address
> 
> An RFC 1413 ident query: the username associated with a client’s TCP connection
> 
> An authenticator: validating the user’s credentials
> 
> A forward DNS lookup: the origin server’s IP address
> 
> An external, user-defined ACL
> 
> Some access rules use fast checks out of necessity. For example, the icp_access rule is a fast check. It must be fast, to serve ICP queries quickly. Furthermore, certain ACL types, such as proxy_auth, are meaningless for ICP queries. The following access rules are fast checks:
> - header_access
> - reply_body_max_size 
> - reply_access 
> - ident_lookup 
> - delay_access 
> - miss_access
> - broken_posts
> - icp_access
> - cache_peer_access
> - redirector_access
> - snmp_access
>
> The following ACL types may require information from external sources (DNS, authenticators, etc.) and are thus incompatible with fast access rules:
>- srcdomain, dstdomain, srcdom_regex, dstdom_regex
>- dst, dst_as
>- proxy_auth
>- ident
>- external_acl_type
> 
>This means, for example, that you can’t reliably use an ident ACL in a header_access rule.




#### Common Scenarios
##### Scenario 1 - Allowing Local Clients Only
> [!info]
> Because access controls can be complicated, this section contains a few examples. They demonstrate some of the common uses for access controls. You should be able to adapt them to your particular needs.
> 
> Allowing Local Clients Only
> Almost every Squid installation should restrict access based on client IP addresses. This is one of the best ways to protect your system from abuses. The easiest way to do this is write an ACL that contains your IP address space and then allow HTTP requests for that ACL and deny all others:
> ```shell
> acl All src 0/0
> acl MyNetwork src 172.16.5.0/24 172.16.6.0/24
> 
> http_access allow MyNetwork
> http_access deny All
> ```
> Most likely, this access control configuration will be too simple, so you’ll need to add more lines. Remember that the order of the http_access lines is important. Don’t add anything after `deny All`. Instead, add the new rules before or after `allow MyNetwork` as necessary.

##### Scenario 2 - Denying Pornography

> [!info]
> Denying Pornography
> ____
Blocking access to certain content is a touchy subject. Often, the hardest part about using Squid to deny pornography is coming up with the list of sites that should be blocked. You may want to maintain such a list yourself, or get one from somewhere else. The “Access Controls” section of the Squid FAQ has links to freely available lists.
> 
> The ACL syntax for using such a list depends on its contents. If the list contains regular expressions, you probably want something like this:
> 
> ```bash
> acl PornSites url_regex "/usr/local/squid/etc/pornlist"
> http_access deny PornSites
> ```
> On the other hand, if the list contains origin server hostnames, simply change url_regex to dstdomain in this example.
#### Ports Known to be Unsafe !

> [!warning]
> Here is an list of the privileged ports that are known to be unsafe:
> ___
> ```bash
> acl Dangerous_ports 7 9 19 22 23 25 53 109 110 119
> acl SSL_ports port 443 563
> acl CONNECT method CONNECT
> 
> http_access deny Dangerous_ports
> http_access deny CONNECT !SSL_ports
> additional http_access lines as necessary...
> ```
> Don’t worry if you’re not familiar with all these strange port numbers. You can find out what each one is for by reading the /etc/services file on a Unix system or by reading IANA’s list of registered TCP/UDP port numbers at http://www.iana.org/assignments/port-numbers.
___

#### Giving special users access
-
![[Pasted image 20241130022638.png]]

![[Pasted image 20241130022658.png]]
___

#### Preventing Cache Hits for Local Sites

> [!tip]
> TIP
The no_cache rules don’t prevent your clients from sending these requests to Squid. There is nothing you can configure in Squid to stop such requests from coming. Instead, you must configure the user-agents themselves.
> 
> If you add a no_cache rule after Squid has been running for a while, the cache may contain some objects that match the new rule. Prior to Squid Version 2.5, these previously cached objects might be returned as cache hits. Now, however, Squid purges any cached response for a request that matches a no_cache rule.
> 

![[Pasted image 20241130023324.png]]

___

#### PORTs & Cache
![[Pasted image 20241130010135.png]]
![[Pasted image 20241130010533.png]]
____

____


#### Testing cofig files eg: `blocked_sites`

> [!light]
> - GET Full Report about Caching and squid overall:
> ```bash
>squidclient mgr:info
>```
> You can easily test some access controls with the squidclient program. For example, if you have a rule that depends on the origin server hostname (dstdomain ACL), or some part of the URL (url_regex or urlpath_regex), simply enter a URI that you would expect to be allowed or denied:
> 
> ```bash
> squidclient -p 4128 http://blocked.host.name/blah/blah
> ```
> or:
> ```shell
> squidclient -p 4128 http://some.host.name/blocked.ext
> ```
> 
> 	Certain aspects of the request are harder to control. If you have src ACLs that block requests from outside your network, you may need to actually test them from an external host. Testing time ACLs may be difficult unless you can change the clock on your system or stay awake long enough.
> 
> You can use squidclient’s `-H` option to set arbitrary request headers. For example, use the following if you need to test a browser ACL.
> 
> ```bash
> squidclient -p 4128 http://www.host.name/blah \
> 	-H 'User-Agent: Mozilla/5.0 (compatible; Konqueror/3)\r\n'
> ```
> _____ 
> 
> For more complicated request, with many headers, you may want to use the technique described in Section 16.4.
> 
> You might also consider developing a routine cron job that checks your ACLs for expected behavior and reports any anomalies. Here is a sample shell script to get you started:
>```bash
>#!/bin/sh
>set -e
> 
>TESTHOST="www.squid-cache.org"
>
># make sure Squid is not proxying dangerous ports
>#
>ST=`squidclient 'http://$TESTHOST:25/' | head -1 | awk '{print $2}'`
>if test "$ST" != 403 ; then
>        echo "Squid did not block HTTP request to port 25"
>fi
> 
> 
># make sure Squid requires user authentication
>
>ST=`squidclient 'http://$TESTHOST/' | head -1 | awk '{print $2}'`
>if test "$ST" != 407 ; then
>        echo "Squid allowed request without proxy authentication"
>fi
>
>
># make sure Squid denies requests from foreign IP addresses
># elsewhere we already created an alias 192.168.1.1 on one of
># the system interfaces
>
>EXT_ADDR=192.168.1.1
>ST=`squidclient -l $EXT_ADDR 'http://$TESTHOST/' | head -1 | awk '{print $2}'`
>if test "$ST" != 403 ; then
>        echo "Squid allowed request from external address $EXT_ADDR"
>fi
> 
>exit 0
>```
> 


___
## Fragments of `squid.conf`

> [!quote]
> My preference is to be conservative and allow only the safe ports. The default squid.conf includes the following Safe_ports ACL:

 - Safe Ports:
```shell
acl Safe_ports port 80          # http
acl Safe_ports port 21          # ftp
acl Safe_ports port 443 563     # https, snews
acl Safe_ports port 70          # gopher
acl Safe_ports port 210         # wais
acl Safe_ports port 1025-65535  # unregistered ports
acl Safe_ports port 280         # http-mgmt
acl Safe_ports port 488         # gss-http
acl Safe_ports port 591         # filemaker
acl Safe_ports port 777         # multiling http

http_access deny !Safe_ports
```

### ACLs Regex
- ident_regex
You saw the ident ACL earlier in this section. The ident_regex simply allows you to use regular expressions, instead of exact string matching on usernames returned by the ident protocol. For example, this ACL matches usernames that contain a digit:
```shell
acl NumberInName ident_regex [0-9]
```
- proxy_auth_regex
As with ident, the proxy_auth_regex ACL allows you to use regular expressions on proxy authentication usernames. For example, this ACL matches admin, administrator, and administrators:
```shell
acl Admins proxy_auth_regex -i ^admin
```


# other subjects



___

Here's a **Squid Cheat Sheet** that covers common Squid commands, configuration options, and useful tips for managing and troubleshooting a Squid proxy server.
> [!tip] ### **Squid - Basic Commands**
> ___
> ___
> - Configure squid at:
> ```bash
> sudo vim /var/log/squid/access.log
>```
>- then restart with:
>```bash
>squid -k restart
>```
> - install at splunk and squid:
> ```bash
> sudo yum install nmap-ncat -y
> ```
> - Test Squid configuration for errors
> ```bash
> sudo squid -k check
> ```
> - Start Squid service
> ```bash
> sudo systemctl start squid
> ```
> - Stop Squid service
> ```bash
> sudo systemctl stop squid
> ```
> - Restart Squid service
> ```bash
> sudo systemctl restart squid
> ```
> - Check the status of Squid service
> ```bash
> sudo systemctl status squid -l
> ```
> - Enable Squid to start on boot
> ```bash
> sudo systemctl enable squid
> ```
> - Disable Squid from starting on boot
> ```bash
> sudo systemctl disable squid
> ```
> - Reload Squid configuration without restarting the service
> ```bash
> sudo squid -k reconfigure
> ```
> - Test Squid configuration for errors
> ```bash
> sudo squid -k parse
> ```
> 
### **Squid Configuration File (`/etc/squid/squid.conf`)**

- **Set HTTP Port (default is 3128)**  
  Change the port Squid listens to for incoming client connections.

  ```bash
  http_port 3128
  ```
  >You can instruct Squid to listen on multiple ports with additional http_port lines. This is often useful if you must support groups of clients that have been configured differently. For example, the browsers from one department may be sending requests to port 3128, while another department uses port 8080. Simply list both port numbers as follows:
```bash
http_port 3128
http_port 8080
```
>You can also use the http_port directive to make Squid listen on specific interface addresses. When Squid is used on a firewall, it should have two network interfaces: one internal and one external. You probably don’t want to accept HTTP requests coming from the external side. To make Squid listen on only the internal interface, simply put the IP address in front of the port number:

```bash
http_port 192.168.1.1:3128
```

____
- CHECK THE TABS BELLOW:
~~~tabs
---tab ACLs Principles
- ACLs - Basics
____

![[Pasted image 20241130002457.png]]
____
![[Pasted image 20241130002517.png]]
____
![[Pasted image 20241130002537.png]]
____
![[Pasted image 20241130002632.png]]
____
- You can also specify hostnames in IP ACLs. For example:

```shell
acl Squid dst www.squid-cache.org
acl Aperture dst aperture.sci
```
> [!tip] - Tip - Squid converts hostnames to IP addresses at startup.
> Once started, Squid never makes another DNS >lookup for the hostname’s address. Thus, Squid never notices if the address changes while it’s >running.

___

>> NEXT TAB >>
____
____
---tab |> More ....
Squid stores IP address ACLs in memory with a data structure known as an splay tree (see http://www.link.cs.cmu.edu/splay/). The splay tree has some interesting self-organizing properties, one of which being that the list automatically adjusts itself as lookups occur. When a matching element is found in the list, that element becomes the new root of the tree. In this way frequently referenced items migrate to the top of the tree, which reduces the time for future lookups.

All subnets and ranges belonging to a single ACL element must not overlap. Squid warns you if you make a mistake. For example, this isn’t allowed:
```shell
acl Foo src 1.2.3.0/24
acl Foo src 1.2.3.4/32
```
It causes Squid to print a warning in cache.log:

> WARNING: '1.2.3.4' is a subnetwork of '1.2.3.0/255.255.255.0'
> WARNING: because of this '1.2.3.4' is ignored to keep splay tree searching
>          predictable
> WARNING: You should probably remove '1.2.3.4' from the ACL named 'Foo'
> In this case, you need to fix the problem, either by removing one of the ACL values or by placing them into different ACL lists.
___

___
___
___
---tab More Content
____
~~~

![[Pasted image 20241130001935.png]]
![[Pasted image 20241130002032.png]]
![[Pasted image 20241130002055.png]]

![[Pasted image 20241130002142.png]]


> [!success] ##### Squid's 3 main cache log files:
> Squid has three main log files: cache.log, access.log, and store.log. The first of these, cache.log, contains informational and debugging messages. When you start Squid the first few times, you should closely watch this file. If Squid refuses to run, the reason is probably at the end of cache.log. Under normal conditions, this log file doesn’t become large enough to warrant any special attention. Also note that if you start Squid with the `-s` option, the important cache.log messages are also sent to your syslog daemon. You can change the location for this log file with the cache_log directive:
> ```shell
> cache_log /squid/logs/cache.log
> ```
> ___
> The access.log file contains a single line for each client request made to Squid. On average, each line is about 150 bytes. In other words, it takes about 150 MB to log one million client requests. Use the cache_access_log directive to change the location of this log file:
>```shell
>cache_access_log /squid/logs/access.log
>``` 
>___
>If, for some reason, you don’t want Squid to log client requests, you can specify the log file pathname as /dev/null.
___
The store.log file is probably not very useful to most cache administrators. It contains a record for each object that enters and leaves the cache. The average record size is typically 175-200 bytes. However, Squid doesn’t create an entry in store.log for cache hits, so it contains fewer records than access.log. Use the cache_store_log directive to change the location:
>```shell
>cache_store_log /squid/logs/store.log
>```
>You can easily disable store.log altogether by specifying the location as none:
> ```shell
> cache_store_log none
> ```
> ___
> 


- **Set Cache Directory**  
  Specify the directory where Squid will store cached content.

  ```bash
  cache_dir ufs /var/spool/squid 10000 16 256
  ```

- **Set Cache Size**  
  Define the maximum size of the cache.

  ```bash
  maximum_object_size 100 MB
  ```

- **Allow or Deny Access Based on IP**  
  Allow or block specific IP addresses or subnets.

  ```bash
  acl allowed_ips src 192.168.1.0/24
  http_access allow allowed_ips
  ```

  To block all other access:

  ```bash
  http_access deny all
  ```

- **Set Up Access Control Lists (ACLs)**  
  Define rules for what content is accessible based on IP, URL, etc.

  ```bash
  acl my_network src 192.168.1.0/24
  acl bad_sites dstdomain .example.com
  http_access deny bad_sites
  ```

- **Redirect URLs or Block Domains**  
  Block or redirect requests to certain domains or URLs.

  ```bash
  acl block_sites dstdomain .example.com
  http_access deny block_sites
  ```

- **Enable Authentication**  
  Enable basic authentication for clients connecting to the proxy.

  ```bash
  auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/squid_passwd
  auth_param basic realm Squid Proxy Server
  acl auth_users proxy_auth REQUIRED
  http_access allow auth_users
  ```

- **Configure Access Logs**  
  Set up the log format for access logs.

  ```bash
  access_log /var/log/squid/access.log squid
  ```

### **Squid Log Files**

- **Access Log Location**:  
  `/var/log/squid/access.log`
  
  This log records client requests, responses, and other proxy interactions.

- **Cache Log Location**:  
  `/var/log/squid/cache.log`
  
  This log records Squid's internal caching activities and errors.

- **Rotate Logs**  
  Log rotation for Squid can be configured through `logrotate`. You may need to add `/etc/logrotate.d/squid` for log rotation configuration.

### **Squid Proxy ACL Examples**

- **Allow a specific IP range access**:

  ```bash
  acl allowed_ips src 192.168.1.0/24
  http_access allow allowed_ips
  ```

- **Block access to a specific website**:

  ```bash
  acl blocked_sites dstdomain .example.com
  http_access deny blocked_sites
  ```

- **Block certain user agents** (e.g., block wget):

  ```bash
  acl bad_user_agents browser -i wget
  http_access deny bad_user_agents
  ```

### **Squid for Logging and Monitoring**

- **Monitor Squid's Cache Usage**:

  ```bash
  squidclient -h localhost -p 3128 mgr:info
  ```

- **View Access Logs in Real-Time**:

  ```bash
  tail -f /var/log/squid/access.log
  ```

- **Check Squid Cache Statistics**:

```bash
sudo squidclient -h localhost -p 3128 mgr:cache_object
```

> [!hint] #### **Squid Proxy Server Troubleshooting**
> 
> - **Check for Configuration Errors**:
> 
> ```bash
> sudo squid -k parse
> ```
> 
> - **Check Squid's Internal Logs**:
> 
> ```bash
> sudo tail -f /var/log/squid/cache.log 
> ```
> - log & cache at once:
> ```bash
> sudo tail -f /var/log/squid/access.log /var/log/squid/cache.log
> ```
> 
> - **Testing Proxy with Curl**:
>   
> ```bash
> curl -x http://your_squid_proxy:3128 http://example.com
> ```

### **Squid Integration with Splunk for Logging**

To forward Squid logs to a Splunk server for centralized logging:

1. **Set up the log format** in `squid.conf` to include detailed logging information.

```bash
access_log /var/log/squid/access.log squid
```

2. **Configure Squid to forward logs to Splunk**:
   - Install Splunk Universal Forwarder on the Squid proxy server.
   - Configure the forwarder to send logs from `/var/log/squid/access.log` to the Splunk server.

  Example command to add the Splunk server as a forward destination:

```bash
sudo /opt/splunkforwarder/bin/splunk add forward-server 10.0.2.148:9997
```

3. **Configure Splunk to receive logs** on port `9997` for TCP input.

```bash
sudo /opt/splunk/bin/splunk enable listen 9997
```

#### **Performance Tuning for Squid**
![[Pasted image 20241130000310.png]]
- **Increase Cache Memory**:  
  If Squid is not performing well, you can adjust the memory cache size.

  ```bash
  cache_mem 512 MB
  ```

- **Tune the Maximum Object Size**:  
  To store larger files in cache:

  ```bash
  maximum_object_size 100 MB
  ```

- **Increase the Number of Cache Objects**:

  ```bash
  cache_dir ufs /var/spool/squid 10000 16 256
  ```


> [!faq] ##### **Squid Authentication**
> 
> - **Basic Authentication**:  
>   To configure Squid to require basic authentication, you can use `ncsa_auth`:
> 
>   ```bash
>   auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/squid_passwd
>   ```
> 
>   You will need to create a password file using the `htpasswd` utility.
> 
> - **Create the Password File**:
> 
>   ```bash
>   htpasswd -c /etc/squid/squid_passwd username
>   ```
> 
### **Squid Caching Directives**

- **Cache Expiration Times**:  
  Set cache expiration times for different types of content.

  ```bash
  refresh_pattern -i \.gif$ 1440 100% 4320
  refresh_pattern -i \.html$ 10 90% 4320
  ```

- **Cache Large Objects for Longer**:

  ```bash
  refresh_pattern -i \.pdf$ 4320 100% 10080
  ```

---

This cheat sheet should help you manage and configure Squid for your environment. If you need additional configuration examples or help with advanced setups, feel free to ask!
#### Squid Ban and Approved list sample
```plaintext
### Sample Allowed Sites
google.com
wikipedia.org
github.com
stackoverflow.com
reddit.com
medium.com
quora.com
coursera.org
udemy.com
khanacademy.org
```
____
```plaintext
### Sample Blocked Sites
forbes.com
yahoo.com
facebook.com
instagram.com
tinder.com
betting.com
4chan.org
chatroulette.com
kik.com 
```
____

> [!info] splunk fowarder
> ![[Pasted image 20241122152648.png]]




> [!tip]  ### - These lists can be used to configure your Squid proxy server, where the allowed sites are accessible and the blocked sites are restricted from access.
> ____
> ___
> Sources
> [1] Visualize the Top Blocked Sites | Zscaler https://www.zscaler.com/blogs/security-research/visualize-top-blocked-sites
> [2] Examples | LeechBlock https://www.proginosko.com/leechblock/examples/
> [3] A Look at the Top Blocked Websites | Zscaler https://www.zscaler.com/blogs/security-research/look-top-websites-blocked
> [4] Squid proxy server - Installation and how to allow access and block websites https://www.youtube.com/watch?v=xfu83JXlBOg
> [5] The List of 65+ Inappropriate Websites to Block for Kids https://findmykids.org/blog/en/list-of-websites-to-block
> [6] About Blocked Sites - WatchGuard https://www.watchguard.com/help/docs/help-center/en-US/Content/en-US/Fireware/intrusionprevention/blocked_sites_about_c.html
> [7] Web filtering block page example text - EduGeek.net https://www.edugeek.net/forums/internet-related-filtering-firewall/176973-web-filtering-block-page-example-text.html
> [8] Distracting Websites to Block at Work (& How to Block Them) https://www.currentware.com/blog/distracting-websites-to-block-at-work/ 
>```

___

> [!summary] ### Objectives
> 
> - Install and configure a Squid proxy server on Ubuntu.
> - Understand basic Squid configuration options.
> - Learn to control access to the proxy server.
> ![[Pasted image 20241121161632.png]]
> <iframe width="560" height="315" src="https://www.youtube.com/embed/lUM6sdYfTIA?si=8p04N7xEufiDMcmw" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
> ### Instructions
> 	 - Default port: 3128
> 	 - can be changed to:
> 	 - 0.0.0.0:3128
> 1. **Install Squid:** Open a terminal and run `sudo apt install squid`.
> 
> 2. **Back up the configuration file:**  Run the following:
>    ```bash
>    sudo cp /etc/squid/squid.conf /etc/squid/squid.conf.original
>    sudo chmod a-w /etc/squid/squid.conf.original
>    ```
> 
> 3. **Configure Squid:** Edit `/etc/squid/squid.conf` using a text editor (e.g., `sudo nano /etc/squid/squid.conf`).
> 
> 4. **Change the TCP port (Optional):**  Locate the `http_port` directive and change the port number if desired. For example, to use port 8888:
>    ```
>    http_port 8888
>    ```
> 
> 5. **Set the hostname (Optional):** Locate the `visible_hostname` directive and set your desired hostname. For example:
>    ```
>    visible_hostname myproxy
>    ```
> 
> 6. **Configure on-disk cache (Optional):** Locate and uncomment the `cache_dir` directive, adjusting the path and size as needed. For example:
>    ```bash
>    cache_dir ufs /var/spool/squid 100 16 256
>    ```
> 
> 7. **Configure access control (Optional):**
>    - **By IP address/subnet:** Add an `acl` rule at the bottom of the ACL section, for example, to allow the subnet `192.168.1.0/24`:
>      ```bash
>      acl mynetwork src 192.168.1.0/24
>      ```
>    - Add an `http_access` rule at the top of the `http_access` section to allow the defined ACL:
>      ```bash
>      http_access allow mynetwork
>      ```
>    - **By time:** Similar to IP access control, define an ACL for time restrictions, such as:
>      ```bash
>      acl workhours time MTWTFS 9:00-17:00
>      ```
>    - Combine ACLs in `http_access` rules.  For example:
>      ```bash
>      http_access allow mynetwork workhours
>      ```
> 
> 8. **Restart Squid:** Run `sudo systemctl restart squid.service`.
> 
___



> [!NOTE] # <u>Steps to Ingest Data from Squid Proxy Logs</u>
> 1. **Locate Squid Logs**: Identify where your Squid logs are stored. By default, they are usually located at `/var/log/squid/access.log`.
> ```plaintext
> /var/log/squid/access.log
> ```

> [!NOTE] # <u>Setting Up a Linux Squid Proxy Server Forwarding to a Splunk Server</u>
> ### Introduction
> In today's digital landscape, managing network traffic efficiently is crucial for organizations. A Squid proxy server, a robust open-source caching proxy, can significantly enhance network performance and security by acting as an intermediary for requests from clients seeking resources from other servers. When integrated with Splunk, a powerful platform for searching, monitoring, and analyzing machine-generated data, it provides comprehensive insights into network traffic and user behavior. This report outlines the steps to set up a Squid proxy server on a Linux system and configure it to forward logs to a Splunk server.

> [!NOTE] # <u>Understanding Squid Proxy</u>
> Squid is a widely-used caching proxy for the web supporting HTTP, HTTPS, FTP, and more. It reduces bandwidth and improves response times by caching and reusing frequently-requested web pages. Squid is highly configurable, allowing administrators to set access controls, authenticate users, and filter content (Linuxize).

> [!NOTE] # <u>Key Features of Squid Proxy</u>
> - **Caching**: Squid stores copies of requested web resources, reducing bandwidth and improving load times.
> - **Access Control**: Administrators can define who can access the internet and what resources they can access.
> - **Content Filtering**: Squid can block access to certain websites or types of content.
> - **Authentication**: Supports various authentication methods, including LDAP and Active Directory integration (Windows OS Hub).

> [!TIP] # <u>Installing Squid Proxy on Linux</u>
> ### Prerequisites
> Before installing Squid, ensure that your Linux system is up-to-date. Use the package manager specific to your Linux distribution to update the system packages.


____

> [!NOTE] # <u>Installation Steps</u>
> Follow these steps to install and configure the Squid proxy server on your Linux system.
>
> **Update Package Lists**: Open the terminal and run the following command to update the package lists:
> ```bash
> sudo apt-get update
> ```
>
> **Install Squid**: Install Squid using the package manager. For Ubuntu, the command is:
> ```bash
> sudo apt-get install -y squid
> ```
>
> **Verify Installation**: Check the Squid version to ensure it is installed correctly:
> ```bash
> squid -v
> ```
>
> **Start and Enable Squid**: Start the Squid service and enable it to run at boot:
> ```bash
> sudo systemctl start squid
> sudo systemctl enable squid
> ```
>
> **Check Status**: Verify that Squid is running:
> ```bash
> sudo systemctl status squid
> ```

> [!NOTE] # <u>Configuring Squid Proxy</u>
> The primary configuration file for Squid is located at `/etc/squid/squid.conf`. Here, you can set up various parameters such as access control lists (ACLs), cache settings, and logging options.
>
> **Access Control**: Define who can access the proxy by editing the ACLs in the configuration file. For example, to allow access from a specific IP range, add:
> ```bash
> acl localnet src 192.168.1.0/24
> http_access allow localnet
> ```
>
> **Cache Settings**: Adjust the cache size and directory:
> ```bash
> cache_dir ufs /var/spool/squid 100 16 256
> ```
>
> **Logging**: Configure logging options to ensure that all requests are logged for analysis. The default log file is `/var/log/squid/access.log`.

> [!NOTE] # <u>Advanced Configuration</u>
> For more advanced configurations, such as SSL bumping or reverse proxy settings, refer to the Squid documentation and community forums for guidance.


> [!NOTE] # <u>Setting Up Splunk</u>
> Install Splunk: Download and install Splunk on your server. Follow the installation guide provided by Splunk for your specific operating system.
> 
> Configure Splunk Inputs: Set up Splunk to receive logs from the Squid server. This involves configuring inputs to listen on a specific port (e.g., 9997) for incoming data.
> 
> Install Splunk Add-on for Squid: Install the Splunk Add-on for Squid Proxy to parse and analyze Squid logs efficiently. This add-on provides pre-built dashboards and reports tailored for Squid logs (Splunk Documentation).

> [!NOTE] # <u>Forwarding Squid Logs to Splunk</u>
> Configure Squid Logging: Ensure that Squid is logging all necessary information. Modify the squid.conf file to set the desired log format.
> 
> Set Up a Forwarder: Install the Splunk Universal Forwarder on the Squid server. This lightweight agent will send logs to the Splunk server.
> 
> Configure Forwarder: Edit the inputs.conf file on the forwarder to specify the log files to monitor and the Splunk server details. For example:
> 
> ```bash
> [monitor:///var/log/squid/access.log]
> 
> index = squid
> 
> sourcetype = squid
> ```
> 
> Start the Forwarder: Start the Splunk Universal Forwarder to begin sending logs to the Splunk server.
> 
> Verify Data in Splunk: Log in to the Splunk web interface and verify that Squid logs are being indexed and available for search and analysis (Splunk Community).

> [!NOTE] # <u>Conclusion</u>
> Setting up a Squid proxy server on a Linux system and integrating it with Splunk provides a powerful solution for managing and analyzing network traffic. Squid's caching and access control features enhance network performance and security, while Splunk's analytics capabilities offer deep insights into user behavior and potential security threats. By following the steps outlined in this report, organizations can efficiently deploy a Squid proxy server and leverage Splunk to optimize their network operations.

## References
Linuxize. (2020, October 23). How to Install and Configure Squid Proxy on Ubuntu 20.04. Linuxize. https://linuxize.com/post/how-to-install-and-configure-squid-proxy-on-ubuntu-20-04/
Its Linux FOSS. How to Install and Configure Squid Proxy on Ubuntu 20.04. Its Linux FOSS. https://itslinuxfoss.com/how-to-install-and-configure-squid-proxy-on-ubuntu-20-04/
Kifarunix. Install and Setup Squid Proxy on Ubuntu 20.04. Kifarunix. https://kifarunix.com/install-and-setup-squid-proxy-on-ubuntu-20-04/
Splunk Documentation. Install the Splunk Add-on for Squid Proxy. Splunk. https://docs.splunk.com/Documentation/AddOns/released/Squid/Install
Splunk Community. Splunk Add-0n for Squid Proxy. Splunk Community. https://community.splunk.com/t5/All-Apps-and-Add-ons/Splunk-Add-0n-for-Squid-Proxy/m-p/555699
Windows OS Hub. How to Install and Configure Squid Proxy Server on Linux. Windows OS Hub. https://woshub.com/install-squid-proxy-server-linux/


_____
```markdown
Some Methods to get the Squid Logs into Splunk
Both Squid and Splunk are awesome, and there are multiple ways of getting logs...
 
            Method One:
Configure Splunk to reach out and take the Squid Logs.
 
            Method Two:
Install the Splunk Forwarder Software Agent onto the Squid Server to Push the Squid Logs towards Splunk.
 
            Method Three:
Configure Squid itself to Push the Squid Logs towards Splunk, and configure Splunk to Listen for them.
Method One and Two are covered in other places , these instructions are for the third method.
 
[ Method Three ]
Steps:
•          Configure the Squid logs to be in a 'better' format to improve Splunk searches.
•          Configure Squid to use this new format both locally and to send them towards Splunk
•          Configure Splunk to Listen for these logs.
 
First - Update the Format of the Squid Logs:
Edit the Squid Configuration File => sudo nano /etc/squid/squid.conf     ( this is the typical squid location, your installation may be in a different folder... )
Scroll to the bottom of the file and add the following lines:
```

```shell
## Preferred Log format for Splunk
logformat splunk_recommended_squid %ts.%03tu logformat=splunk_recommended_squid duration=%tr src_ip=%>a src_port=%>p dest_ip=%<a dest_port=%<p user_ident="%[ui" user="%[un" local_time=[%tl] http_method=%rm request_method_from_client=%<rm request_method_to_server=%>rm url="%ru" http_referrer="%{Referer}>h" http_user_agent="%{User-Agent}>h" status=%>Hs vendor_action=%Ss dest_status=%Sh total_time_milliseconds=%<tt http_content_type="%mt" bytes=%st bytes_in=%>st bytes_out=%<st
```

```bash
## Local Squid Log to now use this new format.
access_log daemon:/var/log/squid/access.log splunk_recommended_squid
```

```bash 
## Send Logs to an external server listening on UDP:514 , where 10.11.12.13 is the IP Address of your Splunk Server.
access_log udp://10.11.12.13:514 splunk_recommended_squid
Now save this file, then restart Squid and verify new log format:
sudo service squid restart
 
sudo service squid status
 
sudo tail -f /var/log/squid/access.log
Second - Configure Splunk to Listen (receive) the sent logs from Squid:
Login the the Splunk , and from the Home Page:
            Click Settings from the Main Menu, the click the Add Data Button
            Scroll down and click on the Monitor Button
From the Left Hand Menu:
Click on TCP / UDP        Click on UDP and type 514 in the Port field - leave the other two fields empty , then at the top click  Next 
Source Type      Click on Select Source Type , and type syslog in the search field
            Scroll down and click on syslog   ( this is the most generic type, as the formatting of the logs are already being sent is a nice format. )
Host     For the Host Method - click on IP , then at the top click  Review 
            Review
Input Type: UDP Port
Port Number: 514
Source name override: N/A
Restrict to Host: N/A
Source Type: syslog
App Context: search
Host: (IP address of the remote server)
Index: default
Now Click  Submit 
Give it a few seconds, then Browse the internet via you Squid Proxy and verify that 'events' are being recorded in Splunk...
```

![[Pasted image 20241120133742.png]]



# rsyslog - Linux [[rsyslog - Linux]] 