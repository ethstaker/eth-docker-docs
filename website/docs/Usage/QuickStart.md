---
title:  Eth Docker QuickStart
sidebar_position: 1
sidebar_label: QuickStart
---

Warnings about the dangers of running Ethereum staking full nodes are in [Recommendations.md](../Support/Recommendations.md).  
In particular, you must be sure to secure your seed phrase, the mnemonic. You need it to recreate keys, and
to set a withdrawal address, if you didn't already do so during key creation.

## Hardware, resource use

Take a look at some [build ideas](../Usage/Hardware.md) and consider clients' [resource requirements](../Usage/ResourceUsage.md)

## Eth Docker QuickStart

For a rapid start, have Ubuntu or Debian Linux installed, and then follow these steps. This has been tested on Debian
11/12 and Ubuntu 22.04/24.04. 

> If you are using a version of Debian with a root password set, please install `sudo` as the `root` user with
> `apt update && apt install sudo git`, then make your non-root user a member of the sudo group with
> `usermod -aG sudo <username>`.

> If you are using a minimal version of Ubuntu or Debian, please install `git` with `sudo apt update && sudo apt install git`

Eth Docker needs to be installed while logged into a user account other than `root`, for security purposes.

Download Eth Docker

`cd ~ && git clone https://github.com/ethstaker/eth-docker.git && cd eth-docker`

Install pre-requisites such as Docker

`./ethd install`

Configure Eth Docker - have an Ethereum address handy where you want Execution Layer rewards to go

`./ethd config`

Start Eth Docker

`./ethd up`

The same script can also be used to stop, start and update the node. Run `./ethd` for a help screen.

> Note that Docker will restart running clients automatically after a reboot. They will remain stopped if you stopped
them with `./ethd stop` or equivalent Docker commands.

All your settings are in `.env` and can be viewed and edited with `nano .env`. You can also re-run `./ethd config` at
any time.

## Next steps

If you are going to run a validating node, [create and import keys](../Usage/ImportKeys.md). 

Forward the [P2P ports](../Usage/Networking.md) that the clients use.

Consider your [Linux security](../Usage/LinuxSecurity.md)

Deposit your 32 ETH using the official [Staking Launchpad](https://launchpad.ethereum.org/en/).

## Advanced use

macOS requires [manual installation](../Usage/Prerequisites.md) of Docker Desktop. 

Explore the sidebar for advanced options. In particular, you can [integrate with RocketPool](../Usage/Advanced/Rocketpool.md),
run an [SSV operator node](../Usage/Advanced/SSV.md), or run an [RPC node](../Usage/Advanced/ClientSetup.md) with either locally
shared RPC/WS ports or these ports [secured by Traefik and https](../Usage/ReverseProxy.md).

## Additional resources

[Youtube Channel](https://www.youtube.com/channel/UCS5mP-iWYxOCBVSVugPYUhQ)  
[Security Best Practices](https://www.coincashew.com/coins/overview-eth/archived-guides/guide-or-how-to-setup-a-validator-on-eth2-mainnet/part-i-installation/guide-or-security-best-practices-for-a-eth2-validator-beaconchain-node)
