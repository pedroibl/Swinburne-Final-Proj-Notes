____

# GitHub free proxy list
>Type on github `free proxies list`
>get an repo with proxy list and go to validate them

#### Validate proxy list at 
>type on google: Free online proxy checker
[Hidemy.Name/proxy-checker](https://hidemy.io/en/proxy-checker/)

>Get proxies with low latency, anonymous with their type eg http and port number
>




### Proxy Providers 
- Free Anonymous Proxy Browser
#### 14 Best Free Proxy Websites
Source: https://www.linkedin.com/pulse/free-proxy-sites-robiul-hossein/
1. CroxyProxy
2. ProxySite
3. Hide Me
4. Hidemyass
5. Hidester
6. Zend2
7. Whoer
8. VPNBook
9. Plain Proxies
10. Proxyium
11. KProxy
12. MegaProxy
13. Steganos free Proxy
14. FilterBypass me

___
### Proxychains

Tutorials: [First of many same youtuber](https://youtu.be/jqrd9Ba3VOc?si=pScYYFxioxdWtADo)
Repo: [Proxychains Official repo](https://github.com/rofl0r/proxychains-ng)
- Instalation on Mac - Homebrew
```bash
brew install proxychains-ng
```

- Ache the `proxychains.conf` file at the path: (If installed using homebrew) and make adjusts
`/opt/homebrew/Cellar/proxychains-ng/4.16/.bottle/etc
and also at:
`/opt/homebrew/etc/proxychains.conf` (Maybe change both)
```bash
vim /opt/homebrew/Cellar/proxychains-ng/4.16/.bottle/etc/proxychains.conf
```

```bash
vim /opt/homebrew/etc/proxychains.conf
```

 - `proxychains.conf` file edited - dynamic_chain uncommented ... and added socks5 at last line
```md
# proxychains.conf  VER 4.x
#
#        HTTP, SOCKS4a, SOCKS5 tunneling proxifier with DNS.


# The option below identifies how the ProxyList is treated.
# only one option should be uncommented at time,
# otherwise the last appearing option will be accepted
#
dynamic_chain
#
# Dynamic - Each connection will be done via chained proxies
# all proxies chained in the order as they appear in the list
# at least one proxy must be online to play in chain
# (dead proxies are skipped)
# otherwise EINTR is returned to the app
#
#strict_chain
#
# Strict - Each connection will be done via chained proxies
# all proxies chained in the order as they appear in the list
# all proxies must be online to play in chain
# otherwise EINTR is returned to the app
#
#round_robin_chain
#
# Round Robin - Each connection will be done via chained proxies
# of chain_len length
# all proxies chained in the order as they appear in the list
# at least one proxy must be online to play in chain
# (dead proxies are skipped).
# the start of the current proxy chain is the proxy after the last
# proxy in the previously invoked proxy chain.
# if the end of the proxy chain is reached while looking for proxies
# start at the beginning again.
# otherwise EINTR is returned to the app
# These semantics are not guaranteed in a multithreaded environment.
#
#random_chain
#
# Random - Each connection will be done via random proxy
# (or proxy chain, see  chain_len) from the list.
# this option is good to test your IDS :)

# Make sense only if random_chain or round_robin_chain
#chain_len = 2

# Quiet mode (no output from library)
#quiet_mode

 Proxy DNS requests - no leak for DNS data
proxy_dns

# method 2. use the old proxyresolv script to proxy DNS requests
# in proxychains 3.1 style. requires `proxyresolv` in $PATH
# plus a dynamically linked `dig` binary.
# this is a lot slower than `proxy_dns`, doesn't support .onion URLs,
# but might be more compatible with complex software like webbrowsers.
#proxy_dns_old

# method 3. use proxychains4-daemon process to serve remote DNS requests.
# this is similar to the threaded `proxy_dns` method, however it requires
# that proxychains4-daemon is already running on the specified address.
# on the plus side it doesn't do malloc/threads so it should be quite
# compatible with complex, async-unsafe software.
# note that if you don't start proxychains4-daemon before using this,
# the process will simply hang.
#proxy_dns_daemon 127.0.0.1:1053

# set the class A subnet number to use for the internal remote DNS mapping
# we use the reserved 224.x.x.x range by default,
# if the proxified app does a DNS request, we will return an IP from that range.
# on further accesses to this ip we will send the saved DNS name to the proxy.
# in case some control-freak app checks the returned ip, and denies to 
# connect, you can use another subnet, e.g. 10.x.x.x or 127.x.x.x.
# of course you should make sure that the proxified app does not need
# *real* access to this subnet. 
# i.e. dont use the same subnet then in the localnet section
#remote_dns_subnet 127 
#remote_dns_subnet 10
remote_dns_subnet 224

# Some timeouts in milliseconds
tcp_read_time_out 15000
tcp_connect_time_out 8000

### Examples for localnet exclusion
## localnet ranges will *not* use a proxy to connect.
## note that localnet works only when plain IP addresses are passed to the app,
## the hostname resolves via /etc/hosts, or proxy_dns is disabled or proxy_dns_old used.

## Exclude connections to 192.168.1.0/24 with port 80
# localnet 192.168.1.0:80/255.255.255.0

## Exclude connections to 192.168.100.0/24
# localnet 192.168.100.0/255.255.255.0

## Exclude connections to ANYwhere with port 80
# localnet 0.0.0.0:80/0.0.0.0
# localnet [::]:80/0

## RFC6890 Loopback address range
## if you enable this, you have to make sure remote_dns_subnet is not 127
## you'll need to enable it if you want to use an application that 
## connects to localhost.
# localnet 127.0.0.0/255.0.0.0
# localnet ::1/128

## RFC1918 Private Address Ranges
# localnet 10.0.0.0/255.0.0.0
# localnet 172.16.0.0/255.240.0.0
# localnet 192.168.0.0/255.255.0.0

### Examples for dnat
## Trying to proxy connections to destinations which are dnatted,
## will result in proxying connections to the new given destinations.
## Whenever I connect to 1.1.1.1 on port 1234 actually connect to 1.1.1.2 on port 443
# dnat 1.1.1.1:1234  1.1.1.2:443

## Whenever I connect to 1.1.1.1 on port 443 actually connect to 1.1.1.2 on port 443
## (no need to write :443 again)
# dnat 1.1.1.2:443  1.1.1.2

## No matter what port I connect to on 1.1.1.1 port actually connect to 1.1.1.2 on port 443
# dnat 1.1.1.1  1.1.1.2:443

## Always, instead of connecting to 1.1.1.1, connect to 1.1.1.2
# dnat 1.1.1.1  1.1.1.2

# ProxyList format
#       type  ip  port [user pass]
#       (values separated by 'tab' or 'blank')
#
#       only numeric ipv4 addresses are valid
#
#
#        Examples:
#
#            	socks5	192.168.67.78	1080	lamer	secret
#		http	192.168.89.3	8080	justu	hidden
#	 	socks4	192.168.1.49	1080
#	        http	192.168.39.93	8080	
#		
#
#       proxy types: http, socks4, socks5, raw
#         * raw: The traffic is simply forwarded to the proxy without modification.
#        ( auth types supported: "basic"-http  "user/pass"-socks )
#
[ProxyList]
# add proxy here ...
# meanwile
# defaults set to "tor"
socks4 	127.0.0.1 9050
socks5  127.0.0.1 9050
```

[[Tor Configuration]]
