---
layout: post
author: d0xb1n
---

# Installing a BIND9 DNS server on Ubuntu

## Step One
### Install Docker

Set up repo's:
```
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

Install docker using:

```sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin```

Double check that it works by trying the `docker` command

Using `sudo usermod -aG docker $USER` you can add youself to the docker group so that you can run the command without needing sudo.

## Step Two
### Create the docker compose file

Create the docker compose file in a folder in your home directory

```
mkdir ~/bind9
touch ~/bind9/docker-compose.yml
```

Add the following using your preferred text editor

```
services:
  bind9:
    image: ubuntu/bind9:latest
    container_name: bind9
    environment:
      - TZ=<INSERT YOUR TIMEZONE HERE>
      - BIND9_USER=root
    ports:
      - "53:53/tcp"          # TCP/UDP for DNS
      - "53:53/udp"      # UDP for DNS
    volumes:
      - ./config:/etc/bind
      - ./cache:/var/cache/bind
      - ./records:/var/lib/bind
    restart: unless-stopped
```
*You need to add your timezone where it says `<INSERT YOUR TIMEZONE HERE>`.*

Create the folders for Bind9. 
```
./config
./cache
./records
```

## Step Three
### Turn off Ubuntu's built in DNS resolver.

```sudo systemctl disable systemd-resolved.service```

You can test if it worked using the `nslookup` command. If it fails, you will be sure that it is not the one being used.

You can now run `docker compose up -d`

## Step Four
### Modify netplan

```nano /etc/netplan/*yournetplan.yaml*```
Replace **yournetplan.yaml** with the name of the file that exists there.

Add the following:

```
network:
    version: 2
    ethernets:
        adapter:
            dhcp4: false
            addresses: [10.0.0.53/24]
            gateway4: 10.0.0.2
            nameservers:
                addresses: [127.0.0.1]
```

Replace **adapter** with the name of your ethernet adapter. You can find it with `ip addr`.

Replace all IP addresses with the IP addresses for your network. (in addresses and gateway)

## Step Five
### Configuring Bind9

Create a `named.conf` file in the config folder.

Add this to it:

```
acl internal {
    10.0.0.0/24;
    127.0.0.0;
    172.18.0.1;
};

options {
    forwarders {
        1.1.1.1;
        1.0.0.1;
    };
    allow-query {
        internal;
    };
};

zone "home.local" IN {
    type master;
    file "/etc/bind/home-local.zone";
};
```

Note that `/etc/bind/` binds to `./config`.

## Step Six
## Creating the zone file

Run `sudo nano ~/bind9/config/home-local.zone` to create and edit the zone file.

Add

```
$TTL    604800
@       IN      SOA     home.local. root.home.local. (
                             1         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL

;
@               IN      NS      home.local.

;
@               IN      A       10.0.0.53

```

Add more A records as needed.
```
;
subdomain       IN      A       192.168.1.10
```

## Step Seven
### Creating Reverse Lookup Zones

Create a new file `reverse.10.0.0.zone` in the folder with the following:

```
$TTL    86400
@       IN      SOA     home.local. root.home.local. (
                             1         ; Serial
                         86400         ; Refresh
                          7200         ; Retry
                        1209600        ; Expire
                         86400 )       ; Negative Cache TTL

;
@               IN      NS      home.local.

;
1               IN      PTR     host1.home.local.
2               IN      PTR     host2.home.local.
3               IN      PTR     host3.home.local.
```

In the `named.conf` file add:

```
zone "0.10.in-addr.arpa" {
    type master;
    file "/etc/bind/reverse.10.0.0.zone";
};
```