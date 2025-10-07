---
title: Web UI
sidebar_position: 9
sidebar_label: Web UI
---

## Lighthouse

The Lighthouse Siren Web UI uses https with self-signed certificates. To use it, add `siren.yml` to `COMPOSE_FILE` in
`.env`, and if you are not going to use traefik, add `siren-shared.yml` as well. It is then reachable on port `2443`
by default. The port can be adjusted in `.env`.

## Prysm

The Prysm Web UI is insecure http, which means an [SSH tunnel](https://www.howtogeek.com/168145/how-to-use-ssh-tunneling/)
should be used to access it if your node is on a cloud VPS.

If you wish to expose the Web UI across the network, which also exposes the key management UI, you can add `prysm-web-shared.yml` to your `COMPOSE_FILE` in `.env`.

If you wish to only expose the Web UI to `localhost` and then use either a browser on the host or an SSH tunnel, you can add `validator-keyapi-localport.yml` to `COMPOSE_FILE` in `.env`.

## Prepare the validator client

The Web UI can be used to import keys and create a wallet, but we also need the password for this
wallet while starting the validator. To get around this chicken-and-egg problem, you can run
`./ethd cmd run --rm create-wallet` now and choose the wallet password you will use during
the Web UI Wallet Creation.

> This password needs to be at least 8 characters long and contain both a number and a special
> character. The script that stores the password here does not enforce that, but the Web UI does.

Either way, once you are done, run `./ethd up` to start the Prysm consensus client
and validator.

## Connect to the Web UI

You need the Web UI secret first. It is shown during startup in the validator client log. You can also run `./ethd keys get-api-token` to get it.

The first time you connect to the Web UI, you'll want to use `http://IP:7500/initialize?token=THETOKEN`, replacing `IP` and `THETOKEN` with the actual IP address and access token.

If your node is on a local, protected LAN, you can access the Web UI on a browser from any machine on your network via the node's IP address, if you use `prysm-web-shared.yml`.

If the node is running on a cloud VPS, you'll want to use `validator-keyapi-localhost.yml` to restrict access to just `localhost`, then open an SSH connection and tunnel the port used by the Web UI.

Example ssh command:
```
ssh -L 7500:<host>:7500 <user>@<host>
```

where `<host>` is the name or IP address of the node.

Placing this into an alias or shell script can make life easier.

Once the SSH tunnel is open, in a browser, open `http://127.0.0.1:7500`. You'll be prompted for a web password,
which doesn't yet exist, and there may be an option to "Create a Wallet" if you did not do so from
command line.

> Note this is insecure http. The SSH tunnel encrypts the traffic.

# Import keys

Assuming you have some `keystore-m` JSON files from `./ethd cmd run --rm deposit-cli-new --execution_address YOURHARDWAREWALLETADDRESS` or some other way of creating Launchpad compatible keys, click on "Create a Wallet".

> These files are in `.eth/validator_keys` if you used the `deposit-cli` workflow. You'll want to move them to the machine you are running the browser on.

Choose to "Import Keystores". Select the `keystore-m` file(s), provide the password to the keystore, choose whether to import slashing protection data, and Continue.

Set the wallet password.  If you chose to store the wallet password with the validator in a previous step,
make sure it matches here: This is the step where you actually create the wallet with that password.

Continue and you will find yourself inside the Web UI, which will show you the consensus client syncing. Once sync is
complete, you will also see validator information.

# Optional: Verify that wallet password was stored correctly

If you chose to start the validator with a stored wallet password, verify that it was stored correctly by running these commands, one at a time:

```
./ethd restart
./ethd logs -f validator
```

You'll need to navigate to the root of the Web UI and log in again after the restart.
