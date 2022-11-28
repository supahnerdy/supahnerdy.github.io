# CYB-3353: Cloud WireGuard VPN (MacOS)

In this guide, we will be installing a VPN connection which we will then test with a few tools. To do this, we will create a DigitalOcean account with an Ubuntu droplet, and install Docker and Wireguard.

In addition to testing the VPN connection on a [PC/Mac](https://www.wireguard.com/install/), we will also test it using a mobile device with the Wireguard app installed. ([App Store](https://apps.apple.com/us/app/wireguard/id1441195209) or [Google Play](https://play.google.com/store/apps/details?id=com.wireguard.android&hl=en_US&gl=US))

The hardware used for this guide is **MacOS 12.5** with an **Intel** chip, as well as an **iOS** device.


# Installation Guide

## 1. DigitalOcean
Firstly, we are going to **create an account on [DigitalOcean](https://m.do.co/c/4d7f4ff9cfe4)**. If you already have an existing account, please create a fresh one for the purpose of this guide. We will need credits in our account so that we can utilize Droplets and setup our VPN server (use the hyperlink!). 

Creating an account may require some verification steps with email. Note that you will be required to **enter a payment method** upon doing so, as your account will automatically be billed at the end of the usage period. Be sure to destroy the account once you no longer need it to prevent any unwanted payments.

Upon account creation, go to the **control panel**, where we will then create our **Ubuntu droplet**.

<img width="1000" alt="digitalocean_cp" src="https://media.discordapp.net/attachments/1046605983734042654/1046606003367587860/image.png">

## 2. Ubuntu 20.04 Droplet

Under Manage, click **Droplets**, then **Create Droplet.** Here you will customize the following settings:
1. Choose an Image: you can pick any Ubuntu version, but to save a few steps, go to Marketplace and select **Docker 20.10.21 on Ubuntu 20.04** or similar (recommended to use Ubuntu 20.04 for familiarity)
2. Choose a Plan: **Basic** shared CPU, **Regular w/ SSD** CPU option, and the **cheapest** monthly option ($6/mo in my case)
4. Choose a datacenter region: pick any from the one **closest to your location**
3. Authentication: SSH keys are **preferred,** as they are more secure. However, if you do not currently have one set up, password will suffice as it is easier.

Leave everything else as default, and press the green **Create droplet** button. When the bar has finished loading, your droplet is ready!

<img width="1000" alt="droplet" src="https://cdn.discordapp.com/attachments/1046605983734042654/1046615307088838666/image.png">

## 3. WireGuard Server

Next, we are going to install a Wireguard Server on our droplet. The following is based on the instructions provided in this guide from theMatrixDev [here](https://thematrix.dev/setup-wireguard-vpn-server-with-docker/). You can SSH into the droplet from your local terminal, but I found it easier to launch a **Droplet Console** directly from the droplet on the website. You can do this under the 'Access' section.

Once you have a working console, make sure that **Docker Compose** is properly installed (Docker itself should already be installed from using the marketplace droplet):
1. `sudo curl -L "https://github.com/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose`
2. `sudo chmod +x /usr/local/bin/docker-compose`



Then, proceed with the following:
1. `mkdir -p ~/wireguard/`
2. `mkdir -p ~/wireguard/config/`
3. `nano ~/wireguard/docker-compose.yml`

In your docker-compose.yml, paste the following content:
```
version: '3.8'
services:
  wireguard:
    container_name: wireguard
    image: linuxserver/wireguard
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Hong_Kong
      - SERVERURL=1.2.3.4
      - SERVERPORT=51820
      - PEERS=pc1,pc2,phone1
      - PEERDNS=auto
      - INTERNAL_SUBNET=10.0.0.0
    ports:
      - 51820:51820/udp
    volumes:
      - type: bind
        source: ./config/
        target: /config/
      - type: bind
        source: /lib/modules
        target: /lib/modules
    restart: always
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    sysctls:
      - net.ipv4.conf.all.src_valid_mark=1
```

You may need to change some settings in the file:
1. `TZ` refers to the timezone; change this according to your location (e.g. America/Chicago)
2. `SERVERURL` refers to the Public IPv4 Address on your droplet. Find this under 'Networking'.

Once your docker-compose.yml is ready, you can start Wireguard with the following:
1. `cd ~/wireguard/`
2. `sudo docker-compose up -d`

Your Wireguard server should be properly built after the console outputs `Creating wireguard ... done`. However, it is a good idea to verify it is up and running with the command `docker ps`.

<img width="1000" alt="wg_server" src="https://cdn.discordapp.com/attachments/1046605983734042654/1046874384675115028/image.png">


## 4. Testing the VPN w/ a Mobile Device
We will now verify that our VPN works on a mobile device. To begin, go to [ipleak.net](https://ipleak.net/) on any web browser on your mobile device and record the current IP address.

<img width="250" alt="ip1" src="https://media.discordapp.net/attachments/1046605983734042654/1046881794571649094/IMG_3706.png?width=505&height=1093">

In the console, we can run the following command which will build an execution log and QR codes for us to use:

`docker-compose logs -f wireguard`

Open the WireGuard app on your mobile device:
1. Click the '**+**' sign 
2. Click **'Create from QR code'**
3. Scan the **QR code** on the Console labeled `PEER phone1 QR code` or similar (allow the app to use your camera)
4. Give the VPN a name (I named it "DigitalOcean VPN") and toggle it on. It should look similar to the following (obviously with nothing blurred out):

<img width="250" alt="ip1" src="https://media.discordapp.net/attachments/1046605983734042654/1046886835328127046/IMG_3708.png?width=504&height=1095">

Now, refresh the ipleak.net website. Your IP address should be now be the same as the **Public IPv4 Address** from your droplet!

<img width="250" alt="ip1" src="https://media.discordapp.net/attachments/1046605983734042654/1046881817187328010/IMG_3707.png?width=505&height=1093">

## 5. Testing the VPN on a PC
For verifying that the VPN works on our PC, we will need to access the configuration file. More than likely, it will be named `peer_pc1.conf`, and be located in `~/wireguard/config/peer_pc1`.

Similarily, we should record the current IP address on our PC:
<img width="1000" alt="ip1" src="https://cdn.discordapp.com/attachments/1046605983734042654/1046889997166465094/image.png">

Now, lets access this .conf file using the console:
1. `cd ~/wireguard/config/peer_pc1`
2. `sudo nano peer_pc1.conf`

You will then need to copy this file onto your PC. 
For MacOS, I did the following:
1. **Copy** the contents of `peer_pc1.conf` from console
2. Open a new File in **TextEdit**
3. Paste the contents and save it as a **".conf"** extension

(NOTE: It may try to append a .txt/.rtf extension at the very end- if this happens just right click the file --> "Get Info" --> remove the unwanted extension from the "Name & Extension" box --> Uncheck "Hide extension")

Finally, open the WineGuard app on your laptop:
1. Click **Import Tunnel(s) from File...**
2. Import the **.conf file** that you just saved
3. Click **Activate**. It should look similar to the following (again, not with things blurred out):

<img width="1000" alt="ip1" src="https://media.discordapp.net/attachments/1046605983734042654/1046897038744363088/image.png?width=1538&height=1094">

And again... when you refresh the ipleak.net website on your laptop, the IP address should be the same as the **Public IPv4 Address** from your droplet!

<img width="1000" alt="ip1" src="https://cdn.discordapp.com/attachments/1046605983734042654/1046896648816705618/image.png">


# Final Words
Thank you for using this tutorial to install a WireGuard VPN using a DigitalOcean droplet w/ Docker on your machine! Hopefully you have some more knowledge about Cloud IaaS's, VPNs, and other services. 

Written by: Tulsano Wibisono


