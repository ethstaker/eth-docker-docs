---
title:  Switching client(s).
sidebar_position: 7
sidebar_label: Switch Clients
---

## Overview

It may be desirable to switch clients if you are using one that is close to a supermajority of validators. If any one
client has a supermajority of the chain, there is a risk that a
[consensus bug could lead to a large inactivity leak](https://www.symphonious.net/2021/09/23/what-happens-if-beacon-chain-consensus-fails/)
and validators on this client would lose almost all of their staked ETH in that worst-case scenario.

Choose any consensus client and any execution client you'd like to use, and then follow the instructions below.

Note that if you change the execution client, you either need sufficient disk space to sync two execution clients, or
accept downtime while your new execution client syncs.

> `sudo` commands for docker are necessary if your user is not part of the `docker` group. If `docker ps` does not succeed, you need to use `sudo` for `docker` or `docker-compose`, or make your user part of the `docker` group.

## Switching to web3signer

With web3signer, the keys do not need to be moved when switching the consensus client. If keys are currently loaded
directly into the validator client, which is the default, they need to be just as carefully moved as when switching
between consensus clients without web3signer.

### 1. Delete validator keys

- Verify that you have the keystore-m files to reimport keys after the switch to web3signer. They should be in `./.eth/validator_keys`.
- Verify that `WEB3SIGNER=false` in .env. If that's not so, **stop** and make it so.
- `./ethd keys delete all`
- Verify keys are gone: `./ethd keys list`

### 2. Enable web3signer

- Edit configuration with `nano .env`, set `WEB3SIGNER=true` and add `:web3signer.yml` to `COMPOSE_FILE`.
- `./ethd up`
- `./ethd logs -f web3signer` and verify it started correctly

### 3. Delete volume for the old validator client

For all but Nimbus and Teku:
- `./ethd stop`
- `docker volume ls` and find the volume that belonged to the old validator client. `docker volume rm` it.
- `./ethd up`

### 4. Reimport validator keys

This carries a slashing risk, take extreme care.

- [ ] Look at https://beaconcha.in/ and verify that the validator(s) you just removed are now
  missing an attestation. Take a note of the epoch the last successful attestation was in.
- [ ] Allow 15 minutes to go by and verify that the last successful attestation's epoch is now
  finalized. Only then take the next step.
- [ ] Verify **once more** that all your validator(s) have been down long
  enough to miss an attestion
- [ ] If you are absolutely positively sure that the old validator client cannot
  start attesting again and 15 minutes have passed / **all** validators'
  last successful attestation is in a finalized epoch, then and only then:
- [ ] Run `./ethd keys import` and import the keys

### 5. Verify that validators are attesting again

Check https://beaconcha.in/ for your validator public keys, as well as the logs of `consensus` and, if not using an
`client-allin1.yml`, `validator` services.

### 6. Reapply any custom per-validator settings

Custom fee recipient, graffiti and gas limit are stored by the validator client. By default, there are no custom
settings. If, however, you used `./ethd keys set-recipient`, `./ethd keys set-gas` or `./ethd keys set-graffiti`
before, you need to do so again after switching to a new validator client.

## Switching only the consensus client, keys in web3signer

### 1. Choose a new consensus client

- Reconfigure the stack, either with `nano .env` or `./ethd config`, and choose a new consensus client and the same execution client. Make sure to choose "checkpoint sync" so the consensus client can sync in minutes.
- Optional: If you set the advanced options `CL_EXTRAS` or `VC_EXTRAS` in `.env`, edit them to conform to the parameters of the new client.
- `./ethd up`
- `./ethd logs -f consensus` and verify it went through checkpoint sync and is following chain head

### 2. Delete volumes for the old consensus client

`docker volume ls` and find the volumes that belonged to the old consensus client, and for all but Nimbus and Teku, corresponding validator client. `docker volume rm` those.

### 3. Re-register validator keys

For Teku, Lighthouse and Lodestar, you will need to manually register the keys that are in web3signer. As the keys
remain in web3signer, this does not carry a slashing risk.

- Run `./ethd keys register`, which will register all keys in web3signer with the new validator client.

Nimbus and Prysm register the web3signer keys automatically on startup, you need not register them manually.

### 4. Verify that validators are still attesting

Check https://beaconcha.in/ for your validator public keys, as well as the logs of `consensus` and, if not using an
`client-allin1.yml`, `validator` services.

### 5. Reapply any custom per-validator settings

Custom fee recipient, graffiti and gas limit are stored by the validator client. By default, there are no custom
settings. If, however, you used `./ethd keys set-recipient`, `./ethd keys set-gas` or `./ethd keys set-graffiti`
before, you need to do so again after switching to a new validator client.

## Switching only the consensus client, keys *not* in web3signer

### 1. Delete validator keys

- Verify that you have the keystore-m files to reimport keys after consensus client switch. They should be in `./.eth/validator_keys`.
- `./ethd keys delete all`

### 2. Choose a new consensus client

- Reconfigure the stack, either with `nano .env` or `./ethd config`, and choose a new consensus client and the same execution client. Make sure to choose "checkpoint sync" so the consensus client can sync in minutes.
- Optional: If you set the advanced options `CL_EXTRAS` or `VC_EXTRAS` in `.env`, edit them to conform to the parameters of the new client.
- `./ethd up`
- `./ethd logs -f consensus` and verify it went through checkpoint sync and is following chain head

### 3. Delete volumes for the old consensus client

`docker volume ls` and find the volumes that belonged to the old consensus client, and for all but Nimbus and Teku, corresponding validator client. `docker volume rm` those.

### 4. Reimport validator keys

This carries a slashing risk, take extreme care.

- [ ] Look at https://beaconcha.in/ and verify that the validator(s) you just removed are now
  missing an attestation. Take a note of the epoch the last successful attestation was in.
- [ ] Allow 15 minutes to go by and verify that the last successful attestation's epoch is now
  finalized. Only then take the next step.
- [ ] Verify **once more** that all your validator(s) have been down long
  enough to miss an attestion
- [ ] If you are absolutely positively sure that the old validator client cannot
  start attesting again and 15 minutes have passed / **all** validators'
  last successful attestation is in a finalized epoch, then and only then:
- [ ] Run `./ethd keys import` and import the keys

### 5. Verify that validators are attesting again

Check https://beaconcha.in/ for your validator public keys, as well as the logs of `consensus` and, if not using an
`client-allin1.yml`, `validator` services.

### 6. Reapply any custom per-validator settings

Custom fee recipient, graffiti and gas limit are stored by the validator client. By default, there are no custom
settings. If, however, you used `./ethd keys set-recipient`, `./ethd keys set-gas` or `./ethd keys set-graffiti`
before, you need to do so again after switching to a new validator client.

## Switching the execution client if downtime is acceptable

### 1. Choose a new execution client

- Reconfigure the stack, either with `nano .env` or `./ethd config`, and choose a new execution client and the same consensus client.
- Optional: If you set the advanced option `EL_EXTRAS` in `.env`, edit it to conform to the parameters of the new client.
- `./ethd up`
- `./ethd logs -f execution` and verify it started sync

### 2. Delete volumes for the old execution client

`docker volume ls` and find the volume that belonged to the old execution client. `docker volume rm` it.

### 3. Wait for sync

Depending on the client, sync takes between an hour and 5 days. Once the execution client is fully synced, your validators will start attesting again.

## Switching the execution client while avoiding downtime

### 0. Verify you have sufficient disk space

`df -h` should you enough space to sync a second execution client. ~1 TiB free is usually good. This may be a tall order on a 2TB drive.

### 1. Change the ports of the existing client stack

So that new and old client do not conflict, in the directory for the existing client stack, `nano .env` and adjust `EL_P2P_PORT` and `CL_P2P_PORT` - if using Prysm in old and new location,
change `PRYSM_PORT` and `PRYSM_UDP_PORT`. "One higher" will work.

Start the existing client stack with `./ethd up`, it will start using the new ports. Peering may be a bit iffy as there is no port forward for the new ports. There is no need to fix that, as it's temporary.

### 2. Create a new client stack

You'll be running a second copy of Eth Docker in its own directory. For example, if the new directory is going to be `~/eth-staker`: `cd ~ && git clone https://github.com/ethstaker/eth-docker.git eth-staker && cd eth-staker` .

Configure the new stack. You can choose the same or a different consensus client, and a different execution client, compared to your existing client stack.
Make sure to choose "checkpoint sync" so the consensus client can sync in minutes. `./ethd config` followed by `./ethd up`.

**Do not** import validator keys yet. Your validators are still running on your old client, and moving them over needs to be done with care to avoid running them in two places and getting yourself slashed.

Observe consensus client logs with `./ethd logs -f consensus`, see that it checkpoint synced. Look at execution client logs with `./ethd logs -f execution` and wait until it is fully synced. Depending on client this takes between
1 hour and 5 days.

### 3. Move your validators

**Exercise extreme caution. Running your validators in two locations at once would lead to slashing**

Follow the [moving a validator](../Support/Moving.md) instructions. You'll be inside the old directory, e.g. `~/eth-docker`, for the first part where you delete the keys and make sure they are gone, and inside the new directory, e.g. `~/eth-staker`, for the second part where you import the keys again.

### 4. Shut down the old client and remove its storage

Inside the directory for the old client, e.g. `cd ~/eth-docker`, remove all storage for the client: `./ethd terminate`.

Finally, you can remove the directory itself: For example, if it was `~/eth-docker`, `cd ~ && rm -rf eth-docker`.

## Switching clients from systemd to Eth Docker

If you are using systemd with guides from Somer Esat, Coincashew or Metanull, and want to give Eth Docker's automation a whirl, these instructions will show you how to make the switch.

### 1. Create a new Eth Docker client stack

Clone Eth Docker, for example into `~/eth-docker`: `cd ~ && git clone https://github.com/ethstaker/eth-docker.git && cd eth-docker` .

Install prerequisites: `./ethd install`

Configure the client stack. "Rapid sync" can let the consensus layer client sync in minutes. `./ethd config`, followed by `./ethd up`.

**Do not** import validator keys yet. Your validators are still running on your old client, and moving them over needs to be done with care to avoid running them in two places and getting yourself slashed.

Observe the consensus client, and make sure it is synchronizing: `./ethd logs -f consensus`.

### 1a. Optional: Move the existing execution client database to its new location

This only works if you've chosen the same execution client in Eth Docker as you are running in systemd. This avoids a fresh sync of the execution client database in the new location. Note you might want to fresh sync if your DB has grown to the point where starting over is beneficial.

In the location for the new client stack, e.g. `~/eth-docker`, stop the stack: `./ethd stop`

`docker volume ls` to see the volume name of the new execution client. For Geth, you might see `eth-docker_geth-eth1-data`.

Remove the partially synced contents of the new database location: `sudo rm -rf /var/lib/docker/volumes/NEWVOLUME/_data`, e,g. for Geth `sudo rm -rf /var/lib/docker/volumes/eth-docker_geth-eth1-data/_data`

Move the directory of the systemd EL database to the new database location. This depends on what execution layer client you are using. This example assumes Geth.

#### Somer Esat's guide
 
Disable the Geth service and move its database: `sudo systemctl disable geth` and `sudo mv /var/lib/goethereum -t /var/lib/docker/volumes/eth-docker_geth-eth1-data`.

#### Metanull's guide

Disable the Geth service and move its database: `sudo systemctl disable geth` and `sudo mv /home/geth/* -t /var/lib/docker/volumes/eth-docker_geth-eth1-data`.

#### Coincashew's guide

Disable the Geth service and move its database: `sudo systemctl disable eth1` and `sudo mv ~/.ethereum -t /var/lib/docker/volumes/eth-docker_geth-eth1-data`.

#### Change permissions and start the stack

Change permissions so Eth Docker can access the Geth database: `sudo chown -R 10001:10001 /var/lib/docker/volumes/eth-docker_geth-eth1-data`

Start the new stack again: `./ethd up`, then observe that your execution client is running well and is synced to head: `./ethd logs -f execution`.

### 1b. If you did not move Geth with 1a: Shut down Geth

This will stop your validators from attesting. You will incur downtime until the new execution layer database has been fully synced.

#### Somer Esat's guide
 
Disable the Geth service and remove its database: `sudo systemctl disable geth` and `sudo rm -rf /var/lib/goethereum`.

#### Metanull's guide

Disable the Geth service and remove its database: `sudo systemctl disable geth` and `sudo rm -rf /home/geth/*`.

#### Coincashew's guide

Disable the Geth service and remove its database: `sudo systemctl disable eth1` and `sudo rm -rf ~/.ethereum`.

### 2. Move your validators

**Exercise extreme caution. Running your validators in two locations at once would lead to slashing**

Make sure you have your `keystore-m_ETC.json` files and the password for them. Ideally, you should also have a `slashing-protection.json` file.

Stop the validator service. Somer Esat: `sudo systemctl stop prysmvalidator`. Metanull and Coincashew: `sudo systemctl stop validator`.

To export the slashing protection DB from Prysm, adjust this command line to your username and location Eth Docker is in and run `sudo validator slashing-protection-history export --datadir=/var/lib/prysm/validator --slashing-protection-export-dir=/home/ubuntu/eth-docker/.eth/validator_keys`

To export the keys from Prysm, run `sudo validator accounts backup --wallet-dir=/var/lib/prysm/validator --backup-dir=/tmp/keys` and then move them to the `eth-docker` directory, again adjusting for your username: `sudo unzip /tmp/keys/backup.zip -d /home/ubuntu/eth-docker/.eth/validator_keys`. Lastly, remove the zip file again: `sudo rm /tmp/keys/backup.zip`
 
#### Somer Esat's guide

First, remove the validator keys from the existing setup. `sudo systemctl stop prysmvalidator`. Remove the keys: `sudo rm -rf /var/lib/prysm/validator`. 

Verify that the validator can't find them: `sudo systemctl start prysmvalidator` and `journalctl -fu prysmvalidator`, observe that it complains it cannot find its keys. **Do not proceed if the keys are still accessible to the validator**

`sudo systemctl stop prysmvalidator` and `sudo systemctl disable prysmvalidator` to shut it down for good.

Follow the [moving a validator](../Support/Moving.md#import-keys-into-new-client) instructions. You already removed the keys: Wait 15 minutes after that to protect against slashing, then import them again from inside the `~/eth-docker` directory.

#### Metanull's guide

First, remove the validator keys from the existing setup. `sudo systemctl stop validator`. Remove the keys: `sudo rm -rf /home/validator/.eth2validators`

Verify that the validator can't find them: `sudo systemctl start validator` and `journalctl -fu validator`, observe that it complains it cannot find its keys. **Do not proceed if the keys are still accessible to the validator**

`sudo systemctl stop validator` and `sudo systemctl disable validator` to shut it down for good.

Follow the [moving a validator](../Support/Moving.md#import-keys-into-new-client) instructions. You already removed the keys: Wait 15 minutes after that to protect against slashing, then import them again from inside the `~/eth-docker` directory.

#### Coincashew's guide

First, remove the validator keys from the existing setup. `sudo systemctl stop validator`. Remove the keys: `sudo rm -rf ~/.eth2validators`.

Verify that the validator can't find them: `sudo systemctl start validator` and `journalctl -fu validator`, observe that it complains it cannot find its keys. **Do not proceed if the keys are still accessible to the validator**

`sudo systemctl stop validator` and `sudo systemctl disable validator` to shut it down for good.

Follow the [moving a validator](../Support/Moving.md#import-keys-into-new-client) instructions. You already removed the keys: Wait 15 minutes after that to protect against slashing, then import them again from inside the `~/eth-docker` directory.

### 3. Remove old beacon database

We can remove the old beacon chain database and disable the service.

#### Somer Esat's guide

`sudo systemctl stop prysmbeacon && sudo systemctl disable prysmbeacon` and `sudo rm -rf /var/lib/prysm`.

Optionally, remove the goeth and prysm users: `sudo deluser goeth && sudo deluser prysmbeacon && sudo deluser prysmvalidator`.

Optionally, remove the old executables: `sudo apt remove -y geth && sudo apt -y auto-remove && sudo rm /usr/local/bin/validator && sudo rm /usr/local/bin/beacon-chain`.


#### Metanull's guide

`sudo systemctl stop beacon-chain && sudo systemctl disable beacon-chain` and `sudo rm -rf /home/beacon/*`.

Optionally, remove the geth and prysm users: `sudo deluser --remove-home geth && sudo deluser --remove-home beacon && sudo deluser --remove-home validator`.

Optionally, remove the old Geth package: `sudo apt remove -y ethereum && sudo apt -y auto-remove`.

#### Coincashew's guide

`sudo systemctl stop beacon-chain && sudo systemctl disable beacon-chain` and `sudo rm -rf ~/prysm`.

Optionally, remove the old Geth package: `sudo apt remove -y ethereum && sudo apt -y auto-remove`.
