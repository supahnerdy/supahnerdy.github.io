# CYB-3353: OpenVAS Docker (MacOS)

In this guide, we will be installing Docker Desktop (and Docker Compose) with an OpenVAS container to run a vulnerability scan. 

The hardware used for this installation is **MacOS 12.5** with an **Intel** chip.


# Installation Guide
The following steps are based off of the OpenVAS repo from [mikesplain](https://github.com/mikesplain/openvas-docker), which has not been maintained for a few years. Alternatively, you may want to install a newer docker image from git user [atomicturtle](https://github.com/Atomicorp/gvm) or the [SCS](https://securecompliance.gitbook.io/projects/).


## 1. Docker Desktop
Docker Desktop already includes Docker Compose in the installation. For installing Docker on Windows, see [here](https://docs.docker.com/desktop/install/windows-install/).

For Mac:
1. Install the correct `.dmg` file based on your CPU chip.
2. Open the `.dmg`, drag it into applications folder, then open the new `.app` file.

> *NOTE: Upon opening Docker, you may get an error message that reads- `You don't have write access to /Users/username/.docker/contexts`*
>
> *To alleviate this, go into Terminal and type: `sudo chown -R username /Users/username/.docker/contexts` (replace username with yours)*


## 2. OpenVAS Container
Run this command in your Terminal to install the container: `docker run -d -p 443:443 --name openvas mikesplain/openvas`

Depending on your bandwidth, it may take some time to install all the NVT's (won't be able to URL into localhost until then). You can monitor this by checking the CPU usage in Activity Monitor.

Eventually, the NVT's will be rebuilt and your Docker should now list the **openvas** container:
<img width="750" alt="openvas_container" src="https://cdn.discordapp.com/attachments/1041772637245935676/1041772658410401822/image.png">


## 3. Run a Vulnerability Scan
To start off, we are just going to run a basic vulnerability scan on an IP address. Normally, you would have to create a new Target and Task in the software, but we will just go through a wizard that will automatically run the process for us. 

1. In your web browser, go to `https://localhost`

> *NOTE: if it gives you a security warning, hit the Advanced button and proceed to the site.*

2. Both username and password are: `admin`
3. In the context menus, go to Scans --> Tasks
4. It may prompt you with a Quick Start Task Wizard. If not, click on the purple magician's wand icon in the top right.
5. Leave the default IP address value and press "Start Scan".

To check that the scan is running, your Activity Monitor should be displaying some CPU usage and processes. Additionally, you can change how often you want the site to refresh while it is doing the scan, which should update the status bar.

When your scan is done, you should be able to see the report under the "Status" section and your graphs should update accordingly:
<img width="750" alt="openvas_tasks" src="https://media.discordapp.net/attachments/1041772637245935676/1041788535000268921/image.png?width=1720&height=1094">


## 4. docker-compose.yml
We'll create a `docker-compose.yml` to provide more configurations to our container.

In Terminal create a new directory and a .yml file in it:
1. `mkdir ~/docker-build` 
2. `cd ~/docker-build`
3. `nano docker-compose.yml`

The following file has the following features of utilizing [nginx](https://www.nginx.com/) and port redirecting. These can be configured to your own needs.

`docker-compose.yml`
```
version: '3'
services:
  nginx:
      image: nginx:alpine
      restart: always
      hostname: nginx
      ports:
        - "80:80"
      links:
        - openvas
      volumes:
        - ./conf/nginx.conf:/etc/nginx/nginx.conf:ro
  nginx_ssl:
      image: nginx:alpine
      restart: always
      hostname: nginx_ssl
      ports:
        # CHANGE for ports
        - "443:443"
      links:
        - openvas
      volumes:
        - ./conf/nginx_ssl.conf:/etc/nginx/nginx.conf:ro
  openvas:
      restart: always
      image: mikesplain/openvas
      hostname: openvas
      expose:
        - "443"
      volumes:
        - "./data/openvas:/var/lib/openvas/mgr/"
      environment:
        # CHANGE for psswd:
        OV_PASSWORD: password123
      labels:
         deck-chores.dump.command: sh -c "greenbone-nvt-sync; openvasmd --rebuild --progress"
         deck-chores.dump.interval: daily
```

Once you are done editing this and any necessary `.conf` files, run `docker-compose up -d`:


<img width="500" alt="openvas_tasks" src="https://media.discordapp.net/attachments/1041772637245935676/1041811746408640613/image.png">


# Final Words

Thanks for using this tutorial to install OpenVAS on your machine! Hopefully you have some more knowledge about Docker, containers, and vulnerability management. 

<img width="750" alt="final_installation" src="https://media.discordapp.net/attachments/1041772637245935676/1041810333565718569/image.png?width=1720&height=1094">

Written by: Tulsano Wibisono


