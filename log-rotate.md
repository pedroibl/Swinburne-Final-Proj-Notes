# Managing logs - Compressing - Rotating - Removing with LogRotate
___

- Intall
```bash
sudo apt-get install logrotate  # On Debian/Ubuntu-based systems
sudo yum install logrotate      # On RHEL/CentOS-based systems
```

- For Squid logs
    - Configure at:
    ```bash
    sudo vim /etc/logrotate.d/squid
    ```
____

- `Logrotate.d` config file sample 1: ( just in case there is a second sample for test )
- At this file you will set rules for your log.
    - How often will the ratation happen
        - Dailly
        - Weekly
        - etc 
        - etc

```bash
/var/log/squid/access.log {
    daily                # Rotate the log daily
    rotate 7             # Keep 7 days of logs
    compress             # Compress old logs to save space
    delaycompress        # Delay compression of previous log (optional)
    notifempty           # Don’t rotate empty logs
    create 0640 squid squid  # Set permissions for new log files
    missingok            # Don’t show an error if log file is missing
    postrotate
        # Restart Squid or reload logs after rotation (depends on your setup)
        /usr/sbin/squid -k rotate
    endscript
}
```
- This file should have perfect indentation otherwise will have errors when implementing.
    - No white spaces
    - No line breaks 

- Run the following to set the new configuration or when editing the log management rules.

```bash
sudo logrotate /etc/logrotate.conf --debug
```
- and Mannully test Logrotate by forcing it `--force`
- It helps to verify if the rules are working as expected.
```bash
sudo logrotate /etc/logrotate.conf --force
```


- Second sample for squid:
```bash
/var/log/squid/*.log {
    weekly              # Rotate logs weekly
    rotate 5            # Keep the last 5 logs
    compress            # Compress old logs to save space
    notifempty          # Do not rotate empty logs
    missingok           # Do not show error if the log file is missing
    nocreate            # Do not create new log files
    sharedscripts       # Ensure postrotate script is only run once
    postrotate
        /usr/sbin/squid -k rotate 2>/dev/null
        sleep 1
    endscript
}
```

____

- rsyslog log management - conf file sample:
- Open /etc/logrotate.d/rsyslog 
```bash
/var/log/syslog {
    weekly            # Rotate logs weekly
    rotate 4          # Keep the last 4 weeks of logs
    compress          # Compress old logs to save space
    notifempty        # Do not rotate empty logs
    missingok         # Do not show error if the log file is missing
    create 0640 root adm  # Create new log files with proper permissions
}

# Repeat the configuration for other log files
/var/log/auth.log {
    weekly
    rotate 12         # Keep the last 12 weeks of logs
    compress
    notifempty
    missingok
    create 0640 root adm
}
```

____

- Splunk log management - Conf file sample

- Create the file:
```bash
sudo vim /etc/logrotate.d/splunk
```
- Sample:
```bash
/opt/splunk/var/log/splunk/*.log {
    weekly                    # Rotate logs weekly
    rotate 4                   # Keep the last 4 rotated logs (you can change this as needed)
    compress                  # Compress old logs to save disk space
    delaycompress             # Delay compression of the previous log until the next rotation
    notifempty                # Do not rotate empty logs
    missingok                 # Do not generate errors if logs are missing
    create 0640 splunk splunk # Permissions for newly created log files
    sharedscripts             # Run postrotate script only once after all files have been rotated

    postrotate
        # Restart Splunk to make it re-open the log files
        /opt/splunk/bin/splunk restart > /dev/null 2>&1 || true
    endscript
}
```

- Aplly Changes:

```bash
sudo logrotate /etc/logrotate.conf --debug
```
- Mannually
```bash
sudo logrotate /etc/logrotate.conf --force
```



