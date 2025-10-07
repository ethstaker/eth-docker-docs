---
title: "Multiple nodes on one host"
sidebar_position: 6
sidebar_label: Multiple copies
---

## Multiple nodes on one host

In this setup, clients are isolated from each other. Each run their own validator client, and if an execution client
is in use, their own execution client. This is perfect for running a single client, or multiple isolated
clients each in their own directory.

If you want to run multiple isolated clients, just clone this project into a new directory for
each. This is great for running testnet and mainnet in parallel, for example.

It can also be used to run the Vero client with multiple client pairs to connect to.

```
cd ~
git clone https://github.com/ethstaker/eth-docker.git <dirname>
```

It is important that `<dirname` is unique, and for simplicity, you may want them all under the same top-level directory.

Works great: `~/eth-main` and `~/eth-hoodi`

Will not work at all: `~/mainnet/eth-docker` and `~/hoodi/eth-docker`. The `<dirname>` is also the Docker Compose project name,
and having two "stacks" with the same name will wreak havoc.
