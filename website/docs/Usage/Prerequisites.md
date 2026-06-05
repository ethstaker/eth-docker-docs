---
title: "Install Prerequisites."
sidebar_position: 12
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

Eth Docker has been tested on currently supported Ubuntu LTS releases.
An [LTS](https://wiki.ubuntu.com/Releases), Long Term Support, version of Ubuntu is recommended, either
Ubuntu Server or Ubuntu Desktop. If installing Ubuntu Server, make doubly sure to extend the "lv", logical volume, to
use your entire disk during the install.

### Automatic installation

Clone the project: `git clone https://github.com/ethstaker/eth-docker.git && cd eth-docker`

Run `./ethd install` and follow prompts

### Manual installation

Install docker-ce.

```
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin docker-buildx-plugin
```

You know it was successful when you saw messages scrolling past that install Docker and Docker Compose.

## Debian Prerequisites

Eth Docker has been tested on currently supported Debian LTS releases.

### Automatic installation

Clone the project: `git clone https://github.com/ethstaker/eth-docker.git && cd eth-docker`

Run `./ethd install` and follow prompts

### Manual installation

Install docker-ce:

```
sudo apt-get update
sudo apt-get -y install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/debian
Suites: $(. /etc/os-release && echo "$VERSION_CODENAME")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin docker-buildx-plugin
```

You know it was successful when you saw messages scrolling past that install docker and docker compose.

```
## Arch Linux Prerequisites

Eth Docker has been tested on Arch Linux with the following setup.

> Arch Linux uses `pacman` as its package manager. The following
> steps differ from Ubuntu/Debian-based installations.

### Manual installation

Install Docker, Docker Compose, and git:

```
sudo pacman -S docker docker-compose git
```

Enable and start the Docker service:

```
sudo systemctl enable docker
sudo systemctl start docker
```

Add your user to the docker group to run Docker without sudo:

```
sudo usermod -aG docker $USER

```

> Log out and back in after adding yourself to the docker group,
> or run `newgrp docker` to apply changes in the current session.

Clone the project: `git clone https://github.com/ethstaker/eth-docker.git && cd eth-docker`

You know it was successful when `docker run hello-world` completes
without errors.

### Known Issues on Arch Linux

**DNS Resolution in Containers:**
Arch Linux uses `systemd-resolved` which can conflict with
Docker's internal DNS, especially on networks with restrictive
DNS policies (e.g. public WiFi). Containers may fail to resolve
external domains even when the host resolves them correctly.

Fix: Add explicit DNS servers to each service in
`docker-compose.yml`:

```yaml
dns:
  - 8.8.8.8
  - 1.1.1.1
```

**Docker Volume Permissions:**
Docker-owned volumes may cause `Permission denied` errors
during git operations. Fix by correcting ownership:

```
sudo chown -R $USER:$USER ./
```

And ensure your data directories are listed in `.gitignore`:

```
geth-data/
lighthouse-data/
```

## Generic Linux

Other distributions are expected to work as long as they support git, Docker, and Docker Compose.

On Linux, Docker Compose runs as root by default. The individual containers do not, they run as local users inside the
containers. "Rootless mode Docker" is limited, but should work to an extent.

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

where `<your external mount point>` is the directory that your other drive is mounted to, e.g. `/mnt/sata`

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

If you are currently running `docker.io` and you'd like to switch to `docker-ce`, the "Community Edition"
released by Docker, Inc., this is how.

You do not need to stop running containers manually, and they will be back up and running after. All volumes and other
data kept in Docker will stay intact.

If you came here because of a nag message, you can switch out `docker.io` for `docker-ce`, or you can narrowly just upgrade
Compose to V2, and remove Compose V1. In that case, stop before "Prepare docker-ce repo".

You can keep `docker.io` on Ubuntu, and add the Compose V2 plugin. This does not work on Debian this way.  
`sudo apt-mark manual docker.io && sudo apt-get remove --autoremove -y docker-compose && sudo apt-get install -y docker-compose-v2`

**Only** if you wish to replace `docker.io` with `docker-ce`, continue below.

Prepare docker-ce repo.

If you are on Ubuntu:  
`export repo=ubuntu`

If you are on Debian:  
`export repo=debian`

```
sudo apt-get update
sudo apt-get install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/${repo}/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/${repo}
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF

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

Eth Docker works with [rootless Docker](https://docs.docker.com/engine/security/rootless/) since version 29.5.0.
IPv4/IPv6 dual-stack works in rootless mode in testing.

To make sure that P2P traffic on the CL and EL works, including incoming peers, and the services start after reboot
without the user logging in, install it like this:
```
sudo apt-get update && sudo apt-get install -y uidmap
sudo systemctl disable --now docker docker.socket
sudo rm -f /var/run/docker.sock
sudo modprobe br_netfilter
echo "br_netfilter" | sudo tee /etc/modules-load.d/br_netfilter.conf
echo "net.ipv4.ip_forward=1" | sudo tee /etc/sysctl.d/10-ip-forward.conf
sudo sysctl --system
mkdir -p ~/.config/docker
echo '{"userland-proxy":false}' >~/.config/docker/daemon.json
dockerd-rootless-setuptool.sh install
sudo loginctl enable-linger $(id -un)
```

If using Grafana, use `grafana-rootless.yml` instead of `grafana.yml`. This omits node-exporter and cadvisor.

If using traefik, either change its ports in `.env` to be above `1024`, or
[expose privileged ports](https://docs.docker.com/engine/security/rootless/#exposing-privileged-ports).

`ufw`, if installed, will control all ports when in rootless mode. Be sure to allow the P2P ports in, UDP and TCP both.

## macOS Prerequisites

> The following prerequisites apply if you are going to use macOS as a server to run an Ethereum staking full node. If
you use macOS to connect *to* a node server, all you need is an SSH client.

- Install [OrbStack](https://orbstack.dev/), e.g. with `brew install orbstack`
- As an alternative to Orbstack, you could install [Docker Desktop](https://www.docker.com/products/docker-desktop) and
allocate 16+ GiB of RAM and 3TB or so of storage space to it, in Preferences->Resources->Advanced. Unlike OrbStack,
Docker Desktop does not dynamically allocate memory and disk space, and it can be temperamental depending on release.
- Install prerequisites via homebrew: `brew install coreutils newt bash gawk`
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
