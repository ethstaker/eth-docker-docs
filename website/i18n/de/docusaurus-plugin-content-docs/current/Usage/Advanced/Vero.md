---
title: "Running Vero with three client pairs"
sidebar_position: 1
sidebar_label: Vero
---

## Architecture

Vero can be used to protect against a black swan event, where the wrong chain justifies or finalizes on Ethereum mainnet,
and validators that signed for this incorrect chain may lose significant value, upwards of several ETH.

The concern is about a correlated, consensus-impacting bug in multiple clients, which together have a supermajority of
the chain. This happened in 2025 on Holesky testnet, for example, with Geth, Nethermind and Besu all having the same bug.

To protect against this, you'd run the Vero validator client, and multiple CL/EL pairs, then require an N of M consensus
between all of them.

I will document this with a 2 of 3 setup here. You should [simulate](https://supermajority.info/simulator) your risk
and adjust this design to the reality of the chain at a given point in time.

High-level, there will be four (4) Eth Docker stacks, potentially all on the same machine, and communicating with each other.

## Client choice

Any combination that protects you against a correlated bug of three (3) clients is good already. If you can get to where
a correlated bug of four (4) clients is survivable, even better. Use the [simulator](https://supermajority.info/simulator).

For this example, I will use Teku/Besu, Nimbus/Reth and Lodestar/Erigon, with a Attestation Consensus Threshold of 2. 

## Vero inna box - all on the same machine

You will need sufficient RAM and storage. On mainnet, 96 GiB RAM and 8TB NVMe should be enough.

### Prepare four (4) Eth Docker stacks

Clone Eth Docker four times, then configure each copy. The directory names should reflect the client mix you use in each,
solely for your convenience.

```
cd ~
git clone https://github.com/ethstaker/eth-docker vero
git clone https://github.com/ethstaker/eth-docker eth-tb
git clone https://github.com/ethstaker/eth-docker eth-nr
git clone https://github.com/ethstaker/eth-docker eth-le
```

### Configure first through third client pair

`<paircode>` here is `tb` for Teku/Besu, `le` for Lodestar/Erigon, &c. Choose something that makes sense to
you and matches the directories you created in the previous step.

```
cd ~/eth-<paircode>
./ethd config
```

- Select Mainnet or Hoodi
- Select "Ethereum RPC node"
- Select Consensus Layer and Execution Layer client
- Select Checkpoint Sync provider
- Say "yes" to MEV Boost and configure relays
- Say "no" to Grafana - this is important
- Configure your fallback fee recipient address

Now edit the `.env` file and change values so this works on one node

`nano.env` and then set

```
COMPOSE_FILE=<what-is-already-here>:ext-network.yml
DOCKER_EXT_NETWORK=vero_default
CL_ALIAS=<paircode>-consensus
EL_ALIAS=<paircode>-execution
MEV_ALIAS=<paircode>-mev
EL_NODE=http://<paircode>-execution:8551
MEV_NODE=http://<<paircode>-mev:18550
CL_P2P_PORT=<adjusted-port>
CL_QUIC_PORT=<adjusted-port>
EL_P2P_PORT=<adjusted-port>
EL_P2P_PORT_2=<adjusted-port>
CL_IPV6_P2P_PORT=<adjusted-port>
```

You need to make sure the P2P ports do not conflict. An easy way to do this would be to add `100` for the second client pair
and `200` for the third client pair. As an example:
- Do not change the P2P ports on the first client pair
- On the second client pair, increase them all by `100`, e.g. `CL_P2P_PORT=9100`
- On the third client pair, increase them all by `200`, e.g. `CL_P2P_PORT=9200`

You'll have better peering when you port forward all P2P ports, most of them both UDP and TCP, with the QUIC port being UDP only.
The intent is to make sure the ports are Internet-reachable.

Also `nano ext-network.yml` and edit it to use `name: vero_default`. This is only necessary until roughly one month after
Pectra, when the `DOCKER_EXT_NETWORK` variable will start to be used.

Repeat this for all three client combinations. Don't start them yet, we need Vero and its network to be up.

What is happening here:
- All three client combos are on the same Docker bridge network, the one Vero uses: `vero_default`. This is done
by appending `ext-network.yml` to the existing `COMPOSE_FILE`, setting `DOCKER_EXT_NETWORK` and, for a limited time,
editing `ext-network.yml`.
- We set aliases so each CL and EL can be reached by a unique name
- We tell the CL which EL it should connect to
- We tell the CL which MEV node it should connect to

### Configure Vero

```
cd ~/vero
cp default.env .env
nano .env
```

Once in the text editor, set these values:
```
COMPOSE_FILE=vero-vc-only.yml:web3signer.yml
WEB3SIGNER=true 
NETWORK=<mainnet or hoodi>
MEV_BOOST=true 
FEE_RECIPIENT=<0xmy-fee-recipient-address>
VC_EXTRAS=--attestation-consensus-threshold 2
CL_NODE=http://<paircode-one>-consensus:5052,http://<paircode-two>-consensus:5052,http://<paircode-three>-consensus:5052
```

Optionally, you can also set a Graffiti:
```
GRAFFITI=<my-very-witty-text>
DEFAULT_GRAFFITI=false
``` 

What is happening here:
- We configure the stack for Vero with Web3signer
- We tell Vero how to reach the CLs, matching the `CL_ALIAS` set earlier. If your client mix is exactly like the example with the
 `tb`, `nr` and `le` pair codes, then it'd read `CL_NODE=http://tb-consensus:5052,http://nr-consensus:5052,http://le-consensus:5052`

### Start clients and import keys

#### Start everything

`cd` to each directory and start, with Vero first - we need its Docker bridge network to be up.

Adjust the client directory names to the ones you actually chose, with their respective pair codes

```
cd ~/vero
./ethd up
cd ~/eth-tb
./ethd up
cd ~/eth-nr
./ethd up
cd ~/eth-le
./ethd up
```

If there are issues here, fix them.

#### Import keys

`cd ~/vero`, then copy your `keystore-m` JSON files into `.eth/validator_keys`

> **Warning** Make extremely sure these keys are not imported anywhere else.

> If they were running somewhere else, ensure that they have been deleted there, you are missing attestations,
> and the last attestion is in a finalized block - usually 15 minutes.

> Failure to heed this warning **will** get your keys slashed - penalized and force-exited.

Once you are certain these keys aren't already running somewhere, import them with `./ethd keys import`

### Verify it all works

Go into each "stack" and check the logs.

For Vero, `./ethd logs -f --tail 50 validator` and `./ethd logs -f --tail 50 web3signer`

For each Ethereum client combo, `./ethd logs -f --tail 50 consensus`
and `./ethd logs -f --tail 50 execution` and `./ethd logs -f --tail 50 mev-boost`.

## Vero on separate machines

Very similar, just do not use aliases or `ext-network.yml`, and use either `cl-shared.yml` or `cl-traefik.yml:traefik-cf.yml`,
see also [port sharing](./SharedPorts.md) and [reverse proxy](../ReverseProxy.md) instructions. Vero would then have a `CL_NODE` to point to each machine's IP address
or if using traefik, to the FQDN of each CL

## Grafana monitoring

This is a little tricky. You don't want multiple copies of Grafana. You'd likely use central-proxy-docker and a centralized
copy of Grafana, either Grafana Cloud (a bit expensive) or self-hosted.

This is a whole rabbit hole in and of itself, and not yet well documented.

