---
title: "Install Prerequisites."
sidebar_position: 11
sidebar_label: Prerequisites
---

Eth Docker relies on Docker and Docker Compose, and git for initial install and updates. It has been tested on Linux,
and is expected to work on MacOS.

> The following prerequisites will be installed on the Linux server you will run your node on. The machine you use to
connect *to* the Linux server only requires an SSH client.

## Ubuntu Prerequisites

> Ubuntu can be installed with the Docker snap package, which can cause issues including complete data loss on upgrade.
We highly recommend removing this via `sudo snap remove --purge docker`. Note this action will delete all data kept
in Docker. If you are running the snap Docker package and have data you need to keep, please ask for help in the
ethstaker Discord.

Eth Docker has been tested on Ubuntu 24.04 "Mantic Minotaur", Ubuntu 22.04 "Jammy Jellyfish" and Ubuntu 20.04
"Focal Fossa". An [LTS](https://wiki.ubuntu.com/Releases), Long Term Support, version of Ubuntu is recommended, either
Ubuntu Server or Ubuntu Desktop. If installing Ubuntu Server, make doubly sure to extend the "lv", logical volume, to
use your entire disk during the install.

### Automatic installation

Clone the project: `git clone https://github.com/ethstaker/eth-docker.git && cd eth-docker`

Run `./ethd install` and follow prompts

### Manual installation

Install docker-ce:

```
sudo apt-get update
sudo apt-get -y install ca-certificates curl gnupg lsb-release whiptail
sudo mkdir -p /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

You know it was successful when you saw messages scrolling past that install Docker and Docker Compose.

If you like, you can also add a docker-compose alias, replacing `MYUSERNAME` with your actual user name:
`echo 'alias docker-compose="docker compose"' >>/home/MYUSERNAME/.profile`

## Debian Prerequisites

Eth Docker has been tested on Debian 11 "Bullseye" and Debian 12 "Bookworm".

### Automatic installation

Clone the project: `git clone https://github.com/ethstaker/eth-docker.git && cd eth-docker`

Run `./ethd install` and follow prompts

### Manual installation

Install docker-ce:

```
sudo apt-get update
sudo apt-get -y install ca-certificates curl gnupg lsb-release whiptail
sudo mkdir -p /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

You know it was successful when you saw messages scrolling past that install docker and docker compose.

If you like, you can also add a docker-compose alias, replacing `MYUSERNAME` with your actual user name:
`echo 'alias docker-compose="docker compose"' >>/home/MYUSERNAME/.profile`

## Generic Linux

Other distributions are expected to work as long as they support git, Docker, and Docker Compose.

On Linux, Docker Compose runs as root by default. The individual containers do not, they run as local users inside the
containers. "Rootless mode" is expected to work for Docker with this project, as it does not use AppArmor.

## Change Docker storage location

Taken from the [RocketPool docs](https://docs.rocketpool.net/guides/node/docker.html#configuring-docker-s-storage-location)

By default, Docker will store all of its container data on your operating system's drive. In some cases, this is
**not** what you want. For example, you may have a small boot drive and a second larger SSD for the chain data.

> If you have just one drive and are good with the default behavior, don't make these adjustments

To change the Docker volume location, create a new file called /etc/docker/daemon.json as the root user:

```
sudo nano /etc/docker/daemon.json
```

Add this as the contents:

```
{
    "data-root": "<your external mount point>/docker"
}
```

where `<your external mount point>` is the directory that your other drive is mounted to.

Next, make the folder:

```
sudo mkdir -p <your external mount point>/docker
```

If you already have existing volumes that you want to move, stop Docker and move them over:

```
sudo systemctl stop docker
sudo cp -rp /var/lib/docker <your external mount point>/
```

Now, restart the Docker daemon so it picks up on the changes:

```
sudo systemctl restart docker
```

After that, Docker will store its data on your desired disk.

## Switching from docker.io to docker-ce

If you are currently running Canonical's docker.io and you'd like to switch to docker-ce, the "Community Edition"
released by Docker, Inc., this is how.

You do not need to stop running containers manually, and they will be back up and running after. All volumes and other
data kept in Docker will stay intact.

If you came here because of a nag message, you can switch out docker.io for docker-ce, or you can narrowly just upgrade
Compose to V2, and remove Compose V1. In that case, stop before "Prepare docker-ce repo".

If you want to keep docker.io, and add the Compose V2 plugin, you can do so by:

`sudo apt-mark manual docker.io && sudo apt-get remove --autoremove -y docker-compose && sudo apt-get install -y docker-compose-v2`

**Only** if you wish to replace docker.io with docker-ce, continue below.

Prepare docker-ce repo:

```
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg lsb-release

sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu  $(lsb_release -cs) stable" \
| sudo tee /etc/apt/sources.list.d/docker.list >/dev/null
 
sudo apt-get update
```

Remove docker.io:

```
sudo apt-get remove --autoremove -y docker.io containerd runc docker-compose docker-compose-v2
```

Reboot - yes this is mandatory:

```
sudo reboot
```

Install docker-ce:

```
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin docker-buildx-plugin
```

Verify that all containers started automatically:
```
sudo docker ps
```

## rootless Docker

Eth Docker works with [rootless Docker](https://docs.docker.com/engine/security/rootless/).

If using Grafana, use `grafana-rootless.yml` instead of `grafana.yml`. This omits node-exporter, cadvisor, promtail and
Loki.

If using traefik, either change its ports in `.env` to be above `1024`, or
[expose privileged ports](https://docs.docker.com/engine/security/rootless/#exposing-privileged-ports).

## macOS Prerequisites

> The following prerequisites apply if you are going to use macOS as a server to run an Ethereum staking full node. If
you use macOS to connect *to* a node server, all you need is an SSH client.

- Install [OrbStack](https://orbstack.dev/), e.g. with `brew install orbstack`
- As an alternative to Orbstack, you could install [Docker Desktop](https://www.docker.com/products/docker-desktop) and
allocate 16+ GiB of RAM and 3TB or so of storage space to it, in Preferences->Resources->Advanced. Unlike OrbStack,
Docker Desktop does not dynamically allocate memory and disk space, and it can be temperamental depending on release.
- Install prerequisites via homebrew: `brew install coreutils newt bash`
- You may need to log out and back into your terminal session to have the right version of bash. Try `bash --version`
and verify it's 5.x or higher.
- Verify git is installed with `git --version`. It will show a Desktop prompt to install it if it isn't.

> An arguably easier path could be to keep macOS just for firmware updates and dual-boot into
[Asahi Linux](https://asahilinux.org/) on Apple Silicon Macs or
[Debian Linux](https://wiki.debian.org/InstallingDebianOn/Apple) on Intel x64 Macs.

## Windows 11

It is technically possible to run Eth Docker on [Windows 11](../Support/Windows.md).

However, the challenges inherent in running on Windows 11 are easier to solve when using the Windows-native versions of
the clients, rather than wrapping Docker around them.

Windows 10/11 is fine as an SSH client to connect *to* your Linux server.
