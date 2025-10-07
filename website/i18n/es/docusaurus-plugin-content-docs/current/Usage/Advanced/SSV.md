---
title: Run an SSV node.
sidebar_position: 3
sidebar_label: Run an SSV node
---

Eth Docker supports running an SSV node, together with a consensus client and execution client of choice.

## Setup Prerequisites
### Get Eth Docker
- Clone this tool
  - `git clone https://github.com/ethstaker/eth-docker.git ssv-node && cd ssv-node`

### On Linux
- Install docker, unless you already have it
  - Run `./ethd install`

### On MacOS
- Install [Docker Desktop](https://www.docker.com/products/docker-desktop) and allocate 16 GiB of RAM and around 3TB
of storage to it
- Install pre-requisites via homebrew
  - `brew install coreutils newt bash`

## Setup an SSV Node

Run `./ethd config` and choose SSV Node in the first dialog:

![eth-docker deployment type dialog](../../../static/img/ssv-node.png)

The next question asks if the operator should be participating in DKG ceremonies. For more information, visit [SSV official documentation on the subject](https://docs.ssv.network/developers/tools/ssv-dkg-client).

![eth-docker deployment type dialog](../../../static/img/dkg.png)

Follow the instructions and finally, choose your preferred consensus and execution clients, and
rapid sync for the consensus client. Choose Grafana for visibility.

The config script will create a config file, password and encrypted node key in `./ssv-config`.

Start everything with `./ethd up`.

You can watch logs with `./ethd logs -f ssv-node`, which will also give you the public key of your node.

Back up the contents of `./ssv-config`! If these are lost, you cannot recreate your node installation as registered
with ssv.network.

>Right after startup, the ssv-node will fail because it cannot get to `http://consensus:5052`. This is normal! It will
resolve once the consensus client has started and is listening on the REST API port. You can use
`./ethd logs -f consensus` to see it do that.

## Mapping ports

By default, the SSV node uses ports TCP 13,001 and UDP 12,001 for its P2P network with other nodes. These ports need to
be reachable from the Internet.

If you need to change the ports, you can do so by changing the `SSV_P2P_PORT` and `SSV_P2P_PORT_UDP` variables in
`.env`, and changing the corresponding values in `ssv-config/config.yaml`.

If you are running the node in a home network, you'll want to [forward these ports](https://portforward.com/router.htm),
then [test the TCP port](https://www.yougetsignal.com/tools/open-ports/). As long as the UDP port forward is set up the
same way, you expect it to work, as well.

## Grafana

Grafana dashboards are included.

Please see the [secure proxy](../../Usage/ReverseProxy.md) docs if you'd like to run Grafana on a secured https port,
rather than insecure 3000.

## Debug logs

SSV writes debug logs into its docker volume. By default, these can be found in the
`/var/lib/docker/volumes/eth-docker_ssv-data/_data` directory. `sudo bash` gets you a root shell that has access.

## Updating

When there is a new version of your execution client, consensus client or of the SSV node, just run `./ethd update`
inside the `ssv-node` directory, which will pull fresh images. Then when you are ready, run `./ethd up` to start using
the new version(s).
