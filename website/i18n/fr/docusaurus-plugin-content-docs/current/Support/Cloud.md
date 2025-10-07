---
title:  Securing a cloud VPS.
sidebar_position: 5
sidebar_label: Cloud Security
---

For the most part, nothing special needs to be done to run Eth Docker on a VPS. However, budget VPS providers do not
filter the traffic that can reach the machine: This is definitely not desirable for unsecured ports like Grafana
or execution client, if the shared option is being used. All that should be reachable are the P2P ports.

The arguably best way to secure Grafana, Web UI and execution client ports is via encryption. For this, please see the [secure proxy](../Usage/ReverseProxy.md)
instructions.

If you prefer to keep the ports unencrypted and wish to secure them via ufw, please read on.
## Securing Grafana and execution client via ufw

While Docker automatically opens the Linux firewall for ports it "maps" to the host, it also
allows rules to be placed in front of that, via the `DOCKER-USER` chain.

The following idea uses that chain and integrates ufw with it, so that simple ufw rules can
be used to secure Grafana.

### 1) Edit after.rules:

`sudo nano /etc/ufw/after.rules` and add to the end of the file, *after* the existing `COMMIT`:

```
*filter
:ufw-user-input - [0:0]
:DOCKER-USER - [0:0]

# ufw in front of docker while allowing all inter-container traffic

-A DOCKER-USER -j RETURN -s 172.17.0.0/16
-A DOCKER-USER -j RETURN -s 172.18.0.0/15
-A DOCKER-USER -j RETURN -s 172.20.0.0/14
-A DOCKER-USER -j RETURN -s 172.24.0.0/13
-A DOCKER-USER -j RETURN -s 192.168.0.0/16

-A DOCKER-USER -j ufw-user-input
-A DOCKER-USER -j RETURN

COMMIT
```

Note this deliberately keeps ufw rules from influencing any traffic sourced from the standard Docker private IP ranges.
This may *not* be what you need, in which case just remove those seven lines, and be sure to allow needed
container traffic through explicit ufw rules, if you are blocking a port.

### 2) Edit before.init

`sudo nano /etc/ufw/before.init` and change `stop)` to read:

```
stop)
    # typically required
    iptables -F DOCKER-USER || true
    iptables -A DOCKER-USER -j RETURN || true
    iptables -X ufw-user-input || true
    ;;
```

Then, make it executable: `sudo chmod 750 /etc/ufw/before.init`

Dropping `ufw-user-input` through `before.init` is a required step. Without it, ufw cannot be reloaded, it would display an error message
stating "ERROR: Could not load logging rules".

### 3) Reload ufw

`sudo ufw reload`

### Example: Grafana on port 3000

Reference [common ufw rules and commands](https://www.digitalocean.com/community/tutorials/ufw-essentials-common-firewall-rules-and-commands)
to help in creating ufw rules.

Say I have Grafana enabled on port 3000 and no reverse proxy. I'd like to keep it reachable via [SSH tunnel](https://www.howtogeek.com/168145/how-to-use-ssh-tunneling/)
while dropping all other connections.

First, verify that Grafana is running and port 3000 is open to world using something like https://www.yougetsignal.com/tools/open-ports/

Next, create ufw rules to allow access from `localhost` and drop access from anywhere else:

- `sudo ufw allow from 127.0.0.1 to any port 3000`
- `sudo ufw deny 3000`

Check again on "yougetsignal" or the like that port 3000 is now closed.

Connect to your node with ssh tunneling, e.g. `ssh -L3000:node-IP:3000 user@node-IP` and browse to `http://127.0.0.1:3000` on the client
you started the SSH session *from*. You expect to be able to reach the Grafana dashboard.

### Example: Prysm Web UI on port 7500

Reference [common ufw rules and commands](https://www.digitalocean.com/community/tutorials/ufw-essentials-common-firewall-rules-and-commands)
to help in creating ufw rules.

Say I have the Prysm Web UI enabled on port 7500 and no reverse proxy. I'd like to keep it reachable via [SSH tunnel](https://www.howtogeek.com/168145/how-to-use-ssh-tunneling/)
while dropping all other connections.

First, verify that Prysm Web UI is running and port 7500 is open to world using something like [you get signal](https://www.yougetsignal.com/tools/open-ports/)

Next, create ufw rules to allow access from `localhost` and drop access from anywhere else:

- `sudo ufw allow from 127.0.0.1 to any port 7500`
- `sudo ufw deny 7500`

Check again on [you get signal](https://www.yougetsignal.com/tools/open-ports/) or the like that port 7500 is now closed.

Connect to your node with ssh tunneling, e.g. `ssh -L7500:node-IP:7500 user@node-IP` and browse to `http://127.0.0.1:7500` on the client
you started the SSH session *from*. You expect to be able to reach the Prysm Web UI.

### Example: Shared or standalone execution client on port 8545

It can be useful to have a single execution client service multiple consensus clients, for example when testing, or running a solo staking docker-compose stack as well as a pool docker-compose stack.

To allow Docker traffic to the execution client while dropping all other traffic:
- `sudo ufw allow from 172.16.0.0/12 to any port 8545`
- `sudo ufw allow from 192.168.0.0/16 to any port 8545`
- `sudo ufw deny 8545`
- `sudo ufw allow from 172.16.0.0/12 to any port 8546`
- `sudo ufw allow from 192.168.0.0/16 to any port 8546`
- `sudo ufw deny 8546`

The rules above are a little overly broad for simplicity, to cover all default Docker subnets. You can restrict this
to the actual defaults by adding more specific rules. For the Docker default subnets, see the section about
`after.rules`.

> With ISP traffic caps, it could be quite attractive to run the execution client in a small VPS, and reference it from a consensus client somewhere
> else. This requires a [secure proxy](../Usage/ReverseProxy.md).

### Allowing Docker traffic to the host IP

Ports mapped to host by Docker are reachable by default without the need for ufw rules. There is one exeption:
If a Docker container on the host tries to reach a port mapped to host by the host IP, this will fail by default.

Example: I am running a Docker container on a host with IP `1.2.3.4`, port `26000` is mapped to host, and the container
tries to reach its own port as `1.2.3.4:26000` instead of `localhost:26000`. This will fail.

This is a highly unusual configuration, as a Docker bridge network would typically be used instead of the host IP.
If you do need to reach the host IP from a Docker container, however, a ufw rule like this would do it:

```
sudo ufw allow from 172.16.0.0/12 to any port <PORT>
sudo ufw allow from 192.168.0.0/16 to any port <PORT>
````

The rules above are a little overly broad for simplicity, to cover all default Docker subnets. You can restrict this
to the actual defaults by adding more specific rules. For the Docker default subnets, see the section about
`after.rules`.

## Acknowledgements

The ufw integration is a slightly tweaked version of https://github.com/chaifeng/ufw-docker by way 
of https://p1ngouin.com/posts/how-to-manage-iptables-rules-with-ufw-and-docker
