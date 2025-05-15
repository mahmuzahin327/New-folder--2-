## Step-by-step instructions for installing DNS Server using Bind9

### Step : 1 Installion Process 
1) Login to your machince with your  `username` and `password` .
2) Check hostname -> hostname
 ```
   Using the command  $hostname to check machince hostname

   $example  -> set-hostname dns.bob.com
 ```
## There are two methods for changing the hostname.
1) Handbook  
2) CLI (Command Line Interface)

### Manual Method  
 ```
Navigate the   file   location    using  `cd/etc/` find         the file name hostname . open  the file  hostname using editor 
nano/hostname  or vim/hostname and change the hostname manually  and same  process  open the file  hosts with nano/hosts.After that  rebort the machine using  'sudo rebort'. your hostname with change  .   
Example  -> London.com -> dns.london.com 

 ```

### Command Line Interface  


 ```
    sudo hostname newhostname -> Change the hostname only for the current session (optional, for short-term updates):  

    Example  - >  sudo hostname  myserver  .

    sudo hostnamectl set-hostname newhostname -> This will make permanent changes to /etc/hostname and /etc/hosts.  

    example -> sudo hostnamectl set-hostname myserver .
    check the  hostname change it or not  by `hostnamectl`
    expected output 
       Static hostname: myserver
        Icon name: computer-vm
        Chassis: vm
        Machine ID: xxxxxxx
        Boot ID: yyyyyyy
        Operating System: Ubuntu 22.04 LTS
        Kernel: Linux 5.x.x
        Architecture: x86-64
 ```
 ## Step 2 Change the   DHCP to static IP 
### Utilize Netplan and Configure the IP  address.
```
Navigate the netplan folder  using ' cd/etc/netplan '

In netplan there is a file  name  `50-cloud-init.yaml`
Edit the file    with  cmd -> sudo nano  50-cloud-init.yaml 
```
## Edit  as  following 
```
```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    ens33:  # Adjust interface name as needed
      dhcp4: no
      addresses: [192.168.190.101/24]  # Change IP for each server
      gateway4: 192.168.190.2
      nameservers:
        addresses: [192.168.190.101, 8.8.8.8]
```
Apply netplan
```
  sudo  netplan apply  

```
##  Installing Bind
```bash
sudo apt update
sudo apt install bind9 bind9utils bind9-doc -y
```
### Configure file  in Bind9

1) Navigate to bind9 folder  -> cd /etc/bind  then ls  
2) In bind9 folder   

```bind.keys  named.conf.default-zones  
db.0       db.empty     named.conf.local          zones.rfc1918
db.127     db.local     named.conf.options
db.255     named.conf   rndc.key
```
### Change file  in folder 

To know which file need to change go to   `nano named.conf`

## Modified
```
"/etc/bind/named.conf.options";
 "/etc/bind/named.conf.local"; 
```
## UnModified 
```
 /etc/bind/named.conf.default-zones"; 
```
## Modified named.conf.options
` named.conf.options is a configuration file for BIND DNS server.`
```bash
options {

              directory "/var/cache/bind";

              #allow-query { localhost; internal-network; };
              listen-on {any;};

              allow-transfer {none; };
              allow-recursion {any;};

              forwarders { 8.8.8.8; };

              recursion yes;

              dnssec-validation auto;
              auth-nxdomain no;

              listen-on-v6 { none; };
};
```
## Checking the  file for error
```bash

named-checkconf /etc/bind/named.conf.options
sudo systemctl restart bind9.service

 ```

### Modified   named.conf.local 
```


Edit zone config:

  ```bash
 zone "bob.com" {
    type master;
    file "/etc/bind/zones/db.bob.com";
};
zone "alice.com" {
    type master;
    file "/etc/bind/zones/db.alice.com";
};
```
Create Zones Directory:
 ```bash
sudo mkdir /etc/bind/zones

sudo nano /etc/bind/zones/db.alice.com
```
### Modified the file 
```bash

$TTL    604800
@       IN      SOA     admin.alice.com. root.admin.alice.com. (
                  202504173     ; Serial (YYYYMMDDNN)
                  604800        ; Refresh
                  86400         ; Retry
                  2419200       ; Expire
                  604800 )      ; Negative Cache TTL;
@       IN      NS      admin.alice.com.
@       IN      A       192.168.190.101
admin   IN      A       192.168.190.101

```
```bash
sudo nano /etc/bind/zones/db.bob.com
```
```bash

$TTL 604800
@       IN      SOA    admin.bob.com. root.admin.bob.com. (
                  2025051402     ; Serial (YYYYMMDDNN)
                  604800         ; Refresh
                  86400          ; Retry
                  2419200        ; Expire
                  604800 )       ; Negative Cache TTL
;
@       IN      NS      admin.bob.com.
@       IN      A       192.168.190.101
admin   IN      A       192.168.190.101

```

Validate and restart:
```bash
named-checkzone matul.com /etc/bind/zones/db.alice.com
named-checkzone gulshan.com /etc/bind/zones/db.bob.com
sudo systemctl restart bind9.service
```

Set Permissions:
```bash
sudo chown -R bind:bind /etc/bind/zones
```
Restart BIND9:
```bash
sudo systemctl restart bind9.service
sudo systemctl enable named
```
---


## Step 4: Testing
### Using dig
```bash
dig  bob.com @192.168.190.101
dig alice.com @192.168.190.101

```

### Using nslookup
```bash
nslookup  bob.com @192.168.190.101
nslookup  alice.com @192.168.190.101
```