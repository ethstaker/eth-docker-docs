---
title: Linux security and setup recommendations
sidebar_position: 6
sidebar_label: Linux Security
---

## SSH key authentication with Linux

This step is vital if your node's SSH port is reachable via the Internet, for example, because
it runs on a VPS.

This step is still recommended if the SSH port is not reachable via the Internet.

For security reasons, you want some form of two-factor authentication for SSH login, particularly if SSH
is exposed to the Internet. These instructions accomplish that by creating an SSH key with passphrase.

To switch to SSH key authentication instead of password authentication, you will start
on the machine you are logging in from, whether that is Windows 10, MacOS or Linux, and then
make changes to the server you are logging in to.

On Windows 10, you expect the [OpenSSH client](https://winaero.com/blog/enable-openssh-client-windows-10/)
to already be installed. If it isn't, follow that link and install it.

From your MacOS/Linux Terminal or Windows Powershell, check whether you have an ssh key. You expect an id_TYPE.pub
file when running `ls ~/.ssh`.

### Create an SSH key pair

Create a key if you need to, or if you don't have `id_ed25519.pub` but prefer that cipher:<br />
`ssh-keygen -t ed25519`. Set a strong passphrase for the key.
> Bonus: On Linux, you can also include a timestamp with your key, like so:<br />
> `ssh-keygen -t ed25519 -C "$(whoami)@$(hostname)-$(date -I)" -f ~/.ssh/id_ed25519`

### macOS/Linux, copy public key

 If you are on macOS or Linux, you can then copy this new public key to the Linux server:<br />
`ssh-copy-id USERNAME@HOST`

### Windows 10/11, copy public key

On Windows 10/11, or if that command is not available, output the contents of your public key file
to terminal and copy, here for `id_ed25519.pub`:<br />
`cat ~/.ssh/id_ed25519.pub`

On your Linux server, logged in as your non-root user, add this public key to your account:<br />
```
mkdir ~/.ssh
nano ~/.ssh/authorized_keys
```
And paste in the public key.

### Test login and turn off password authentication

Test your login. `ssh user@serverIP` from your client's MacOS/Linux Terminal or Windows Powershell should log you in
directly, prompting for your key passphrase, but not the user password.

If you are still prompted for a password, resolve that first. Your ssh client should show you errors in that case. You
can run `ssh -v user@serverIP` to get more detailed output on what went wrong.

On Windows 10 in particular, if the ssh client complains about the "wrong permissions" on the `.ssh` directory or
`.ssh/config` file, go into Explorer, find the `C:\Users\USERNAME\.ssh` directory, edit its Properties->Security, click
Advanced, then make your user the owner with Full Access, while removing access rights to anyone else, such as SYSTEM
and Administrators. Check "Replace all child object permissions", and click OK. That should solve the issues the
OpenSSH client had.

Lastly, once key authentication has been tested, turn off password authentication. On your Linux server:<br />
`sudo nano /etc/ssh/sshd_config`

Find the line that reads `#PasswordAuthentication yes` and remove the comment character `#` and change it to `PasswordAuthentication no`.

And restart the ssh service, for Ubuntu you'd run `sudo systemctl restart ssh`.

## Set Linux to auto-update

Since this system will be running 24/7 for the better part of 2 years, it's a good idea to have it patch itself.
Enable [automatic updates](https://libre-software.net/ubuntu-automatic-updates/) and install software so the
server can [email you](https://caupo.ee/blog/2020/07/05/how-to-install-msmtp-to-debian-10-for-sending-emails-with-gmail/).

For automatic updates, `"only-on-error"` mail reports make sense once you know email reporting is working and
if you choose automatic reboots, trusting that your services will all come back up on reboot. If you'd like
to keep a closer eye or schedule reboots yourself, `"on-change"` MailReport is a better choice.

For msmtp, I followed the instructions as-is.

## Time synchronization on Linux

The blockchain requires precise time-keeping. On Ubuntu, systemd-timesyncd is the default to synchronize time,
and [chrony](https://en.wikipedia.org/wiki/Network_Time_Protocol) is an alternative.

systemd-timesyncd uses a single ntp server as source, and chrony uses several, typically a pool. The default shipping with Ubuntu can get
out of sync by as much as 600ms before it corrects. My recommendation is to use chrony for better accuracy.

For Ubuntu, install the chrony package. This will automatically remove systemd-timesyncd. Chrony will start automatically.<br />
`sudo apt update && sudo apt -y install chrony`

Check that chrony is synchronized: Run `chronyc tracking`.

> If you wish to stay with systemd-timesyncd instead, check that `NTP service: active` via 
> `timedatectl`, and switch it on with `sudo timedatectl set-ntp yes` if it isn't. You can check
> time sync with `timedatectl timesync-status --all`.

## Firewalling

You'll want to enable a host firewall. You can also forward the P2P ports of your execution and consensus
clients for faster peer acquisition.

Docker will open execution and consensus client P2P (Peer to Peer) ports and the Grafana port automatically. Please make sure the Grafana port cannot be reached directly. If you need to get to Grafana remotely,
an [SSH tunnel](https://www.howtogeek.com/168145/how-to-use-ssh-tunneling/) is a good choice.

For a VPS/cloud setup, please take a look at notes on [cloud security](../Support/Cloud.md). You'll want to
place ufw "in front of" Docker if you are using Grafana or a standalone execution client without a reverse proxy,
and if your cloud provider does not offer firewall rules for the VPS.

Ports that I mention can be "Open to Internet" can be either forwarded to your node if behind a home router, or allowed in via the VPS firewall.

> Opening the P2P ports to the Internet is optional. It will speed up peer acquisition, which
> can be helpful. To learn how to forward your ports in a home network, first verify
> that you are [not behind CGNAT](https://winbuzzer.com/2020/05/29/windows-10-how-to-tell-if-your-isp-uses-carrier-grade-nat-cg-nat-xcxwbt/).
> Then look at [port-forwarding instructions](https://portforward.com/) for your specific router/firewall.

Forward only the ports that you actually use, depending on your client choices.

- 30303 tcp/udp - Geth/Nethermind/Besu/Erigon execution client P2P. Open to Internet.
- 9000 tcp/udp - Lighthouse/Teku/Nimbus/Lodestar/Prysm consensus client P2P. Open to Internet.
- 443 tcp - https:// access to Grafana and Prysm Web UI via traefik. Open to Internet.
- 22/tcp - SSH. Only open to Internet if you want to access the server remotely. If open to Internet, configure
  SSH key authentication.

On Ubuntu, the host firewall `ufw` can be used to allow SSH traffic.

Docker bypasses ufw and opens additional ports directly via "iptables" for all ports that are public on the host,
which means that the P2P ports need not be explicitly listed in ufw.

* Allow SSH in ufw so you can still get to your server, while relying on the default "deny all" rule.
  * `sudo ufw allow OpenSSH` will allow ssh inbound on the default port. Use your specific port if you changed
    the port SSH runs on.
* Check the rule you created and verify that you are allowing SSH, on the port you are running it on.
  You can **lock yourself out** if you don't allow your SSH port in. `allow OpenSSH` is sufficient
  for the default SSH port.
  * `sudo ufw show added`
* Enable the firewall and see numbered rules once more
  * `sudo ufw enable`
  * `sudo ufw status numbered`

> There is one exception to the rule that Docker opens ports automatically: Traffic that targets a port
> mapped by Docker, where the traffic originates somewhere on the same machine the container runs on,
> and not from a machine somewhere else, will not be automatically handled by the Docker firewall rules,
> and will require an explicit ufw rule.
> Steps to allow for this scenario are in [cloud security](../Support/Cloud.md)


## Additional and recommended Linux performance tuning

### noatime

By default, Linux will write a new file timestamp on every read. As you may imagine, this is no bueno for database applications like an Ethereum node.

You can increase the lifetime of your SSD - and incidentally get a small speed boost - by turning this "atime" feature off.

`sudo nano /etc/fstab`

Find the entry for your `/` filesystem, or, if you moved the docker `data-root`, the file system docker lives on.

Find the 4th column. It might read `defaults` right now. Append `,noatime` to it. A full entry might look something like this:

`/dev/disk/by-uuid/33162132-f374-417d-817e-04fdd77e5e11 / ext4 defaults,noatime 0 1`

Don't delete any parameters, just add `,noatime`. And make sure to add that to the 4th column, not anywhere else.

Save, and test with `sudo mount -o remount /`. If that completes without errors, you got it right.

### swappiness

By default, Linux will use swap **a lot**. And, yep you guessed it, that ain't great for database applications like an Ethereum node.

So let's set [swappiness](https://www.howtogeek.com/449691/what-is-swapiness-on-linux-and-how-to-change-it/) to `1`.

`sudo nano /etc/sysctl.conf`

Scroll to the bottom of this file and add

`vm.swappiness=1`

Close and save. Then load the new value with

`sudo sysctl -p`

Alternatively, if you have 32 GiB of RAM or more, you can disable swap entirely with

`sudo swapoff -a`

then edit `/etc/fstab` with `sudo nano /etc/fstab` and comment out the swap volume(s).

### Side channel mitigations - CAUTION

**Here be dragons**

On a VM, VPS, cloud instance, &c, leave this alone. Do not turn off mitigations. They exist for a reason, to keep other processes on the same CPU from reading your secrets.

If however this is a machine you own - baremetal or solo node at home - and the only thing running on here is the Ethereum node, you can turn off side channel mitigations for a small speed boost.

`sudo nano /etc/default/grub`

Find `GRUB_CMDLINE_LINUX` and add `  mitigations=off`

Close and save. `sudo update-grub` and then `sudo reboot`. Provided you got the edit right, the system will come back up, and you can check vulnerability mitigations are now off with `lscpu`

Once more, **don't do this** if the physical machine is shared with VMs or processes you do not control.

## Non-root user on Linux

A standard Ubuntu or Debian install already has a non-root user, that is any user not actually named `root`.

If you are on a VPS that only gives you `root` and do not have a non-root user already, create a non-root user
with your `USERNAME` of choice to log in as, and give it sudo rights. `sudo` allows you to
run commands `as root` while logged in as a non-root user.

```
adduser USERNAME
```

You will be asked to create a password for the new user, among other things. Then, give the new user
administrative rights by adding it to the `sudo` group.

```
usermod -aG sudo USERNAME
```

Optional: If you used SSH keys to connect to your Ubuntu instance via the `root` user you
will need to [associate the new user with your public key(s)](#ssh-key-authentication-with-linux).

## Optional: User as part of docker group

Optionally, you may want to avoid needing to type `sudo` every time you run a docker command. In that
case, you can make your local user part of the `docker` group.

> Please note that a user that is part of the docker group has `root` privileges through docker,
> even without the use of `sudo`. You may not want this convenience trade-off and choose to
> explicitly use `sudo` with all docker commands instead.

```
sudo usermod -aG docker USERNAME
```

followed by

```
newgrp docker
```

## Set up IPMI

This step is highly hardware-dependent. If you went with a server that has IPMI/BMC - out of band management of
the hardware - then you'll want to configure IPMI to email you on error.
