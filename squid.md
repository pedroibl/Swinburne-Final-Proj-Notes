### Objectives

- Create a custom Access Control List (ACL) in Squid proxy server.
- Deny access to specific websites using the ACL.
- Demonstrate configuration and troubleshooting of Squid.

### Instructions

1. Open the Squid configuration file (`squid.conf`).
2. Locate the `http_access` section in the file.
3. Add the following lines _before_ any existing `http_access` rules:
   ```
   acl block_websites dstdomain .msn.com .espn.com
   http_access deny block_websites
   ```
   - Replace `.msn.com .espn.com` with the domains you want to block. Separate multiple domains with spaces.
4. Save the configuration file.
5. Restart the Squid service:
   - Stop Squid: `sudo pkill -9 squid` (or the appropriate command for your system).
   - Start Squid: `sudo squid` (or the appropriate command for your system).
6. Verify that the blocked websites are inaccessible and that access to other sites remains unaffected.
7. If you encounter errors:
   - Check the Squid log files for specific error messages.
   - Carefully review the configuration file for typos or syntax errors, paying close attention to ACL names and references. The error message will often indicate the line number with the problem.

---

- Squid server
  sudo yum update -y
  sudo yum upgrade -y

- sudo vim /etc/squid/squid.conf

sudo tail -f /var/log/squid/access.log
sudo tail -f /var/log/squid/cache.log

sudo squid -k parse
sudo systemctl restart squid
sudo systemctl status squid.service -l

---

##### ACLs

# Define an ACL for allowed sites

acl allowed_sites dstdomain "/etc/squid/allowed_sites.txt"

# Deny all other sites

http_access allow allowed_sites

http_access deny all

For blocking websites:

# Define an ACL for blocked sites

acl blocked_sites dstdomain "/etc/squid/blocked_sites.txt"

# Deny access to those sites

## http_access deny blocked_sites

# Install Splunk

- start updating the instance:

sudo yum update -y

sudo yum upgrade -y

Run the following command to install the RPM package:
1 -
sudo rpm -i splunk-console.rpm
2 -
sudo /opt/splunk/bin/splunk start --accept-license
3 -
sudo /opt/splunk/bin/splunk enable boot-start
4 -
sudo /opt/splunk/bin/splunk status

- Splunk Credentials:
  Username:  Admin
  Password:  Adminpa55

- Config file:
- sudo /opt/splunk/bin/splunk/
- sudo vim /etc/squid/allowed_sites
- sudo chmod +x /etc/squid/allowed_sites
- sudo vim /etc/squid/blocked_sites
- sudo chmod +x /etc/squid/blocked_sites

Restart the server

sudo systemctl status squid.service

sudo squid -k parse

sudo systemctl restart squid

sudo tcpdump -vv -i eth0 udp port 514

-

## Server configuration file

sudo vim /opt/splunk/etc/system/local/server.conf
ls -

---

# udp port

[udp://514]
connection_host = ip
sourcetype = squid
index = main

# point where to grab log

[monitor:///var/log/squid/access.log]
sourcetype = squid
index = main::

---
