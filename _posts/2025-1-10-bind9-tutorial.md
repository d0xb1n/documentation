---
layout: post
author: d0xb1n
---

# Installing a BIND9 DNS server on Ubuntu

## Step One
### Install Docker

Install docker using:

`sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin`

Double check that it works by trying the `docker` command

Using `sudo usermod -aG docker $USER` you can add youself to the docker group so that you can run the command without needing sudo.

## Step Two
### Create the docker compose file

Create the docker compose file in a folder in your home directory

`touch ~/bind9/docker-compose.yml`

Add the following using your preferred text editor

```

```

## Step Three
Turn off Ubuntu's built in DNS resolver.

`sudo systemctl disable systemd-resolved.service`

You can test if it worked using the `nslookup` command
