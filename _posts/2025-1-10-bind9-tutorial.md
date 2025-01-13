---
layout: post
author: d0xb1n
---

# Installing a BIND9 DNS server on Ubuntu

## Step One
### Install Docker

Install docker using:

```sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin```

Double check that it works by trying the `docker` command

Using `sudo usermod -aG docker $USER` you can add youself to the docker group so that you can run the command without needing sudo.

## Step Two
### Create the docker compose file

Create the docker compose file in a folder in your home directory

```touch ~/bind9/docker-compose.yml```

Add the following using your preferred text editor

```

```

## Step Three
### Turn off Ubuntu's built in DNS resolver.

```sudo systemctl disable systemd-resolved.service```

You can test if it worked using the `nslookup` command. If it fails, you will be sure that it is not the one being used.

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
            gateway: 10.0.0.2
            nameservers:
                addresses: [127.0.0.1]
```

Replace **adapter** with the name of your ethernet adapter. You can find it with `ip addr`.

Replace all IP addresses with the IP addresses for your network. (in addresses and gateway)