___
| [[Squid Proxy Server]] | [[DNS - Open DNS - Mac Linux]] | [[Web Deployment - DNS Records]]
| [[Splunk Trainning]] | [[Splunk and  SplunkForwarder]] 

- Install splunk through `.rpm` installler
```bash
rpm -ivh <splunInstallFIle.rpm>
```

> [!info] #### Logs pipeline Management - Analyzing Log files. 
> ___
> ___
> ___
> - You start analysing log files in 3 steps.
> - You will:
> 	- 1  Processing Rules - It allow me to extract, redact, and copy fields from my events Log.
> 	- 2 Metricization Rules - It reduce noise by turning my log in Metrics so can be quantified
> 	- 3 Infinite Logging Rules - Archive selected logs to AWS S3 bucket while indexing some in Splunk

![[Pasted image 20241121194728.png]]

> [!hint] ### - Splunk CLI Sheet Sheet
> ___
> ___
> ___
> 
> - make it start when the server starts 
> ```bash
> sudo /opt/splunk/bin/splunk enable boot-start
> ```
> - Get splunk's status:
> ```bash
> sudo systemctl status squid.service
> ```
> ```bash
> sudo /opt/splunk/bin/splunk list udp
>```
> - tcp
>```bash
>sudo /opt/splunk/bin/splunk list tcp
>```
> <br>
> ___
> ___
> ___ 
> ___
>
> <br>
> 
Create inputs.conf at: 
> ```bash
> sudo touch  /opt/splunk/etc/system/local/inputs.conf
> 
> sudo vim  /opt/splunk/etc/system/local/inputs.conf
> ```
> 
>  - File `inputs.conf` - Create an udp entry and point where to monitor
>```bash
>[udp://514]
>disabled = false
>index = squid
>sourcetype = squid:access:recommended
>
>[monitor:///var/log/squid/access.log]
>disabled = false
>index = squid
>sourcetype = squid:access:recommended
>
>[monitor:///opt/splunk/var/log/splunk/tracker.log]
>disabled = false
>index = squid
>sourcetype = squid:access:recommended
>```
> - Visualize if the file monitor is being displayed with the following command
>```bash
> sudo /opt/splunk/bin/splunk list monitor
>```
> - Generate Diagnostic Bundle:
>```bash
> sudo /opt/splunk/bin/splunk diag
>```

___

~~~tabs
---tab File:  `inputs.conf` | 
>  - File `inputs.conf` - Create an udp entry and point where to monitor
>  - Create the `inputs.conf` at: `/opt/splunk/etc/system/local/`
>  ```bash
>  sudo vim /opt/splunk/etc/system/local/inputs.conf
>  ```
>  ___
>  
> ```shell
> [udp://514]
> connection_host = ip
> sourcetype = squid
> index = main
> 
> [monitor:///var/log/squid/access.log]
> sourcetype = squid
> index = main
> ```
---tab Commands | 
> - make it start when the server starts 
> ```bash
> sudo /otp/splunk/bin/splunk enable boot-start
> ```
> - Get splunk's status:
> ```bash
> sudo systemctl status squid.service
> ```
---tab Check the Splunks File installation
```bash
# For MD5
md5sum /path/to/splunk_installer_file

# For SHA256
sha256sum /path/to/splunk_installer_file
```
~~~

~~~tabs
---tab tab1
- Content tab 1
- lorem
---tab tab2
> [!info] - Content of the tab 2
~~~

> [!check] #### More splunk CLI commands:
> ___
> - Add an TCP or UDP Remote server:
>```bash
>sudo /opt/splunk/bin/splunk add remotehost ip-10-0-0-236.ec2.internal:514
>```



> [!info] ### Adding and configuring Indexes through CLI
> ___
> - Add an Index: > example:
>```bash
> sudo /opt/splunk/bin/splunk add index squid 
>```
> - List all indexes
>```bash
> sudo /opt/splunk/bin/splunk list index
>```
> - Check all indexes configuration - If exists
>```bash
> sudo vim /opt/splunk/etc/system/local/index.conf
>```

  ____
 
> [!todo] ### Splunk SSL configuration sample at `server.conf` 
> - Server configuration file  >> SSL warnings
> ```bash
> sudo vim /opt/splunk/etc/system/local/server.conf 
> ```
> - add the lines at the SSL configuration at `server.conf`
> ```shell
>sslVerifyServerCert = true
>```
> - Sample Configuration
> The warning about "Server Certificate Hostname Validation is disabled" indicates that Splunk is not verifying the hostname of the server certificate when establishing SSL connections. This is a security consideration, and you may want to enable hostname validation by configuring the server.conf file. You can find this setting under the [sslConfig] stanza:
>```plaintext
>[sslConfig]
sslPassword = $7$NK5Xb5xsIhWUV9Id8QccXW68l5JDPjqImd8VtxI9A43QJ3scKWxj0Q==
sslVerifyServerCert = true
cliVerifyServerName = true
>```
>

___ 
> - Start splunk
> ```bash
> sudo splunk/bin/splunk start
> ```
> OR 
> if its installed at `/otp`
> ```bash
> sudo /opt/splunk/bin/splunk restart
> ```
> 
> ```bash
> sudo /opt/splunk/bin/splunk stop
> ```
> 
> ```bash
> sudo /opt/splunk/bin/splunk status
> ```
> - make it start when the server starts 
> ```bash
> sudo /opt/splunk/bin/splunk enable boot-start
> ```
> 
> ```plaintext
> http://localhost:8000
> ```
> 
> - maybe that path to find a file that will monitor 
>  ```bash
>  cd /opt/splunk/etc/apps/userName/local
> ```
> 
> ![[Pasted image 20241112094034.png]]


![[Pasted image 20241112094509.png]]