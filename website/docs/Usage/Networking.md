---
title: Networking and port forwarding
sidebar_position: 5
sidebar_label: Network configuration
---

## Static IP

With a VPS, you have a static IP by default. In your home network, that is typically not the case.

You'll want a static IP address for your server, one that doesn't change. This allows easier
port-forwarding for your Ethereum client peering, and easier management and remote access for you:
You do not need to find a changing server address. You can do set an unchanging IP address a few
different ways.

- You can configure your router to use a [DHCP reservation](https://homenetworkadmin.com/dhcp-reservation/).
How to do this depends on your router.
- You could instead choose an IP address *outside* your DHCP range and [configure it as a static IP](https://linuxhint.com/setup_static_ip_address_ubuntu/).

In Ubuntu Desktop this is done through Network Manager from the UI, and in Ubuntu Server you'll handle it
from CLI via netplan. Check your router configuration to see where your DHCP range is, and what
values to use for default gateway and DNS.

## Port forwarding

By default, the clients use UDP/TCP 30303 and UDP/TCP 9000 as their P2P, "Peer to Peer", ports. These should be "open to Internet". If you're on a home network, set up [port forwarding](https://portforward.com/) on your router for these. If you are behind CGNAT from your ISP, this will not work. To verify, look at the WAN IP of your router and at https://www.whatismyip.com/. If they match, you are not behind CGNAT. If they don't, you are. Ask your ISP for a static IP address to resolve this.

## Firewalling

In a home network, nothing special is required for firewalling. You can use UFW and allow only OpenSSH, if you like.

If you are with a cloud provider and they do not offer a firewall, **and** you chose to use Grafana for monitoring, you'll want to make sure Grafana is firewalled so it is only reachable via SSH tunnel or traefik. Please see [Cloud Security](../Support/Cloud.md), how to set up a [secure proxy](../Usage/ReverseProxy.md) and this [Youtube walkthrough](https://www.youtube.com/watch?v=F2dL7j-sEHY&t=4s).
