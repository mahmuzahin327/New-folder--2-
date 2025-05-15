## Step 1: Set hostname
```bash
hostnamectl set-hostname dns.bob.com
hostname
````
## Step 2: Setting Static IP
Configure IP using Netplan:
```bash
cd  /etc/netplan/
cd  /ect/netplan/50-clound-init.yaml
`then`
cd /