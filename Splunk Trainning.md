___
| [[Splunk]]

```bash
scp ~/.ssh/id_ed25519.pub plunk@192.168.2.128:/home/splunk/.ssh/
```

```bash
scp ~/.ssh/id_ed25519.pub plunk@192.168.2.128:/home/splunk/.ssh/authorized_keys
```

- `ssh-copy-id` command `-i`
```bash
ssh-copy-id -i ~/.ssh/SSH_KEY.pub plunk@192.168.2.128:/home/splunk/.ssh/
```

