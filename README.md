# personal-network
Config for personal VPN + pihole

## What?

Simple docker-based configuration for PiHole and OpenVPN.

## Why?

Pihole grants network-level ad-blocking and pushes the computations for it onto a dedicated server. This should effective replace ublock origin on desktop, while adding that functionality to all other devices (mobile, etc)

OpenVPN is an open source VPN server+client. Normally, I wouldn't use any vpn on desktop, and I'd use the free google VPN on mobile. I'd rather not have Google snoop on my internet activity either, so I've set up my own for personal+family use.

## How?

### Server-Side

Pretty simple, this is meant to run on a simple VPS instance. AWS Lightsail is pretty cheap ($5/month) and I'm already invested in the AWS ecosystem, so that's where I'll be running this.

First, I installed some common tools (mosh, neovim, docker, etc.) + cloned the repo.

#### PiHole

* Open DNS ports from the lightsail console (and note the public IP)
* disable the built in systemd dns resolution service:
  - https://discourse.pi-hole.net/t/docker-reply-from-unexpected-source/5729/3
  - commands:
    ```bash
    sudo systemctl disable systemd-resolved.service
    sudo service systemd-resolved stop
    sudo rm /etc/resolv.conf
    ```
* add host to `etc/hosts` to silence "unknown host (none)" when `sudo`-ing
  - https://askubuntu.com/questions/59458/error-message-sudo-unable-to-resolve-host-none
* start up pihole: `docker-compose up -d pihole`
* Note the admin password: `docker-compose logs pihole | grep random`
* try logging into the admin console. The easist way is to just forward the port over ssh, then load it locally:
  - `ssh -L 8080:localhost:80 lightsail`
  - open `localhost:8080/admin` in your browser
  - login + poke around

#### OpenVPN

* follow the quick start here: https://github.com/kylemanna/docker-openvpn/blob/master/docs/docker-compose.md
  - Once done, you shoulld have a CA + base config file + a client certificate
* modify the dns configuration within the config file to use the pihole as the dns
* open up the VPN port (1194 UDP)
* bring up the service: `docker-compose up -d openvpn`
* connect from a client (make sure to pull off the generated `*.opvn` file)

### Clients

#### Router

I don't currently have a vpn-capable router, so I just set the DNS on the router to the pi-hole public IP. This should at least block ads + trackers on local devices not connected to the VPN

#### OS X Desktop

Download the open source TunnelBlick (https://tunnelblick.net/) and import the config file. That should be all you need to do.

#### Android

Download the OpenVPN android app (https://play.google.com/store/apps/details?id=de.blinkt.openvpn&hl=en_US) and import the generated client config
