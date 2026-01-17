---
title: IPv6
sidebar_position: 10
sidebar_label: IPv6 support
---

## Conquer network amxiety

As of mid 2024:

CGNAT is becoming more prevalent, making it harder to make P2P ports available inbound and get good peering.

Docker support for IPv6 is better.

Ethereum clients are starting to add v6 support, which can benefit from testing.

## How then

### Check you have v6

You can test, from a PC on your LAN, that [whatismyip](https://whatismyip.com) shows both a public v6 and v4 address.
If so, your router is set correctly. This will likely be the case "out of the box" on any ISP that uses CGNAT, and on
several that don't, like Comcast in the US.

On your node, `ip address show` should show you both a v4 `inet` and public v6 `inet6` address on the interface. If
all you see is an `fe80::` address for inet6, you don't have a routable v6 address configured, and you'll want to fix
that, first.

### Configure Docker

You'll want to be on the latest version of docker-ce, at least `27.0.1`.

If you previously had IPv6-specific settings in `/etc/docker/daemon.json`, you can remove them now.

### Configure Eth Docker

`nano .env` and set `IPV6=true`, which will tell Docker Compose to enable v6 for the networks it creates. Then
`./ethd restart` to recreate the bridge network Eth Docker uses.

### Verify

Look at Eth Docker's default network with `docker network ls` and then `docker network inspect eth-docker_default`,
or whatever network it actually is. If IPv6 is enabled, you'll see that and you will see assigned IPv4 and IPv6
addresses.

## Security

`ufw` integration works; ports mapped to host can be blocked by a v6 deny rule. As with v4, ufw needs to be
["in front of Docker"](../Support/Cloud.md) for this to work.

Note that host-mapped ports may not be reachable by default if ufw has been enabled. If you use ufw, please
place it ["in front of Docker"](../Support/Cloud.md) and configure explicit allow rules for your P2P ports.

On your LAN firewall, if this is in a LAN, you'd need rules to allow the P2P ports incoming to the v6 address of your
node.

## Which clients?

This is a moving target and best tracked by [Sonic's site](https://ipv6eth.info). As of mid 2024, all CLs with
the exception of Prysm support a v4/v6 dual stack, and ELs have added support as well.
