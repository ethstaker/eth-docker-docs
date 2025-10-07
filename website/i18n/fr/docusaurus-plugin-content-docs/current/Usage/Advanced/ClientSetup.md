---
title: "Manual setup"
sidebar_position: 7
sidebar_label: Manual setup
---

## Client choice - manual setup

> You can refer back to the [Overview](/) to get a sense of how the
> validator client, consensus client and execution client are
> connected to each other, and which role each plays.

Please choose:

* The consensus client you wish to run
  * Lodestar
  * Nimbus
  * Teku
  * Grandine
  * Lighthouse
  * Prysm
  * Caplin - built into Erigon
* Your execution client you wish to run
  * Reth
  * Besu
  * Nethermind
  * Geth
  * Erigon
* Whether to run a grafana dashboard for monitoring

First, copy the environment file.  
`cp default.env .env`

> This file is called `.env` (dot env), and that name has to be exact. docker compose
will otherwise show errors about not being able to find a `docker-compose.yml` file,
which this project does not use.
 
Then, adjust the contents of `.env`. On Ubuntu Linux, you can run `nano .env`.

- Set the `COMPOSE_FILE` entry depending on the client you are going to run,
and with which options. See below for available compose files. Think of this as
blocks you combine: One consensus client, optionally one execution client, optionally reporting,
optionally a reverse proxy for https:// access to reporting.
- Set the `NETWORK` variable to either "mainnet" or a test network such as "goerli"
- Set the `GRAFFITI` string if you want a specific string.
- If you are going to run a validator client only, without a consensus client, set `CL_NODE` to the URL of your
Ethereum PoS beacon, and choose one of the `CLIENT-vc-only.yml` entries in `COMPOSE_FILE`.
- If you are going to send statistics to https://beaconcha.in, set `BEACON_STATS_API` to your API key
- If you want to sync the consensus client quickly, set `RAPID_SYNC_URL` to a checkpoint provider such as checkpointz
- Adjust ports if you are going to need custom ports instead of the defaults. These are the ports
exposed to the host, and for the P2P ports to the Internet via your firewall/router.

## Compose files

The main concept to understand is that all files in `COMPOSE_FILE` inside `.env` are combined in order, and Docker
Compose will then act on the resulting config. It can be viewed with `docker compose config`.

Set the `COMPOSE_FILE` string depending on which client you are going to use. Add optional services with `:` between
the file names.

Choose one consensus client:

- `teku.yml` - Teku
- `lodestar.yml` - Lodestar
- `nimbus.yml` - Nimbus
- `lighthouse.yml` - Lighthouse
- `prysm.yml` - Prysm

Choose one execution client:

- `reth.yml` - Reth
- `besu.yml` - Besu
- `nethermind.yml` - Nethermind
- `erigon.yml` - Erigon execution client
- `geth.yml` - Geth execution client

> If you wish to use the built-in Caplin consensus client with Erigon, use `erigon.yml` without a consensus client file,
and it will use the built-in Caplin consensus client

Optionally, enable MEV boost or Commit boost:

- `mev-boost.yml` - add the mev-boost sidecar
- `commit-boost-pbs.yml` - add the cb-pbs sidecar

Optionally, choose a reporting package:

- `grafana.yml` - Enable local Grafana dashboards
- `grafana-cloud.yml` - Run a local Prometheus with support for remote-write to Grafana Cloud

- `grafana-shared.yml` - to map the local Grafana port (default: 3000) to the host. This is not encrypted and should
not be exposed to the Internet. Used *in addition* to `grafana.yml`, not instead. Using encryption instead via
`traefik-*.yml` is recommended.
- `prysm-web-shared.yml` - to map the Prysm web port (default: 3500) to the host. This is not encrypted and should
not be exposed to the Internet. Using encryption instead via `traefik-*.yml` is recommended.
- `siren.yml` - Lighthouse's Siren UI

> See [Prysm Web](../../Usage/WebUI.md) for notes on using the Prysm Web UI

Optionally, add ethdo for beacon chain queries:

- `ethdo.yml` - add Attestant's ethdo tool for querying your consensus layer aka beacon node

Optionally, make the staking-deposit-cli available:

- `deposit-cli.yml` - Used to generate mnemonics and signing keys. Consider running key generation offline, instead,
and copying the generated `keystore-m` files into this tool.

Optionally, add encryption to the Grafana and/or Prysm Web pages:

- `traefik-cf.yml` - use encrypting secure web proxy and use CloudFlare for DNS management
- `traefik-aws.yml` - use encrypting secure web proxy and use AWS Route53 for DNS management
- `el-traefik.yml,` `cl-traefik.yml`, `ee-traefik.yml` - advanced use, use traefik for access to execution RPC,
consensus REST and execution engine RPC API ports respectively. Be very cautious with these, always
[firewall](../../Support/Cloud.md) that access to trusted source IPs.

With these, you wouldn't use the `-shared.yml` files. Please see [Secure Web Proxy Instructions](../../Usage/ReverseProxy.md)
for setup instructions for either option.

For example, Teku with Besu:
`COMPOSE_FILE=teku.yml:besu.yml`

## Specialty yml files

Eth Docker supports some specialty use cases. These are the corresponding yml files.

- `ext-network.yml` - Connect to another Docker network, for example for reverse hybrid with RocketPool or connecting
to a central traefik/prometheus.
 - `central-metrics.yml` - Scrape metrics from a
[central prometheus](https://github.com/CryptoManufaktur-io/central-proxy-docker)
- `nimbus-stats.yml` - Send Nimbus stats to beaconcha.in app
- `prysm-stats.yml` - Send Prysm stats to beaconcha.in app
- `ssv.yml` - Run an SSV DVT node
- `contributoor.yml` - EthPandaOps Contributoor, for use alongside a consensus layer client. Read more at [Contributoor GitHub](https://github.com/ethpandaops/contributoor)