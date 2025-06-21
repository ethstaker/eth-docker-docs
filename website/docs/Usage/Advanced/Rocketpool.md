---
title: Integration with Rocketpool.
sidebar_position: 2
sidebar_label: Integration with Rocketpool
---

RocketPool node and Eth Docker solo staking, it's like peanut butter and jelly. Both use docker, and they integrate seamlessly.

I'll be showing two configurations: Chain databases in Rocketpool, and chain databases in Eth Docker.

This should work for any client that Rocketpool supports.

## Configuration A: Chain databases in RocketPool

For this example, the consensus client (beacon/eth2) and execution client (eth1) will run in RocketPool, and Eth Docker will run just a validator client, but not "consensus" or "execution" containers.

### Configure RocketPool

If you are not running RocketPool already, install it [following their instructions](https://docs.rocketpool.net/guides/node/docker.html). When you get to the step where you configure RocketPool:

`rocketpool service config` and choose Locally Managed, any local Execution Client (Geth, Erigon, etc) and any Consensus Client.

Run `rocketpool service start`, and everything should come up. 

You can continue following the Rocketpool instructions at this point.

### Configure Eth Docker

If you are not running Eth Docker already, grab it with `git clone https://github.com/ethstaker/eth-docker.git && cd eth-docker`.

#### With remote Prysm

Configure Eth Docker with `./ethd config`. Make sure to choose the same Ethereum PoS network as RocketPool, and a "Validator client only" to match the Consensus client in RocketPool. Choose "eth2:5053" (note no "http://") as your "remote consensus client" if your solo validator client is Prysm, and "http://eth2:5052" as your "remote consensus client" if your solo validator client is anything else. Doppelganger protection works for Prysm and Nimbus VCs towards a Prysm CL. Any other VC requires Doppelganger to be off.

#### For any other remote CL

Configure Eth Docker with `./ethd config`. Make sure to choose the same Ethereum PoS network as RocketPool, and a "Validator client only" to match the Consensus client in RocketPool. Choose "http://eth2:5052" as your "remote consensus client".

> Lighthouse and Teku are mutually compatible, they can be mixed and matched

#### Start and cleanup

If you had previously already imported keys to Eth Docker, restart Eth Docker with `./ethd restart`.

Optional cleanup: If you had chain databases in Eth Docker previously, do a `docker volume ls` and then `docker volume rm` the consensus/beacon and eth1/ec volumes, e.g. `eth-docker_geth-eth1-data` and `eth-docker_lhbeacon-data`.

### Import solo staking keys

If you haven't already imported validator keys to Eth Docker, do so now. You will need the `keystore-m` files and their
password.

- Copy `keystore-m` files to `.eth/validator_keys` in the `eth-docker` directory
- From the `eth-docker` directory, start Eth Docker: `./ethd up`
- Import keys and follow prompts: `./ethd keys import`
- Verify keys came in: `./ethd keys list`

### Check logs

You expect `./ethd logs -f validator` to show a successful connection. 

`rocketpool service logs validator` should show the validator connecting to its own consensus client.
`docker ps` should show `rocketpool_eth1` and `rocketpool_eth2` containers, but no `consensus` or `execution` containers for Eth Docker.

## Configuration B: Chain databases in Eth Docker

For this example, the consensus client (beacon/eth2) and execution client (eth1) will run in Eth Docker, as well as the solo staking validator client, and RocketPool will run its own validator client as well as its node container, but not "eth1" or "eth2" containers.

### Configure RocketPool

If you are not running RocketPool already, install it [following their instructions](https://docs.rocketpool.net/guides/node/docker.html). When you get to the step where you configure RocketPool:

- `rocketpool service config` and choose `Externally Managed` for the Execution Client, and as your http URL use `http://execution:8545` and as the websocket URL use `ws://execution:8546`. Choose `None` for your fallback client. Choose `Externally Managed` for the Consensus Client, and choose the client you want to run in Eth Docker. As the http URL use `http://consensus:5052`. If you are using Prysm, use `consensus:4000` instead.

Set up Eth Docker next before following the rest of the Rocketpool instructions.

### Configure Eth Docker

If you are not running Eth Docker already, grab it with `git clone https://github.com/ethstaker/eth-docker.git && cd eth-docker`, and configure it with `./ethd config`. Choose an Ethereum node deployment. Make sure to choose the same Ethereum PoS network and consensus layer client as you chose in RocketPool.

Of note, the Prysm validator client in RocketPool only works with the Prysm consensus layer client in Eth Docker. Other clients can be mixed and matched to an extent, e.g. a Lighthouse validator client in RocketPool to a Teku consensus layer client in Eth Docker.

Connect Eth Docker to RocketPool's docker network.

- `nano .env` and add `:ext-network.yml` to `COMPOSE_FILE`
- `nano ext-network.yml` and change the line that reads `name: traefik_default` to `name: rocketpool_net`
- `./ethd start` or, if you already have Eth Docker running, `./ethd update` followed by `./ethd up`
- `rocketpool service start` and Rocketpool should come up

You can continue following the Rocketpool instructions at this point.

### Import solo staking keys

If you haven't already imported validator keys to Eth Docker, do so now. You will need the `keystore-m` files and their
password.

- Copy `keystore-m` files to `.eth/validator_keys` in the eth-docker directory
- From the eth-docker directory, start Eth Docker: `./ethd up`
- Import keys and follow prompts: `./ethd keys import`
- Verify keys came in: `./ethd keys list`

### Check logs

`./ethd logs -f consensus` should show it starting up, ditto `./ethd logs -f execution`, and after consensus is up, you expect
`./ethd logs -f validator` to show a successful connection. Solo staking is (still) working.

`rocketpool service logs validator` should show the validator connecting to your consensus client.
`docker ps` should show no `rocketpool_eth1` or `rocketpool_eth2` containers.
