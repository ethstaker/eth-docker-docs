---
title:  Supported Clients.
sidebar_position: 3
sidebar_label: Clients
---

This project builds from client teams' official docker images or from official source repositories, pulled
directly from docker hub or github, respectively. In most cases, binary is the default.

Currently supported consensus clients:
- [Lodestar](https://github.com/ChainSafe/lodestar)
- [Nimbus](https://github.com/status-im/nimbus-eth2)
- [Teku](https://github.com/Consensys/teku)
- [Grandine](https://github.com/grandinetech/grandine)
- [Lighthouse](https://github.com/sigp/lighthouse)
- [Prysm](https://github.com/prysmaticlabs/prysm)

Currently supported execution clients:
- [Reth](https://github.com/paradigmxyz/reth)
- [Besu](https://github.com/hyperledger/besu)
- [Nethermind](https://github.com/NethermindEth/nethermind)
- [Geth](https://github.com/ethereum/go-ethereum)
- [Erigon](https://github.com/ledgerwatch/erigon)
- [Ethrex](https://github.com/lambdaclass/ethrex)
- [Nimbus EL](https://github.com/status-im/nimbus-eth1)

> An Ethereum node has one consensus client and one execution client. Eth Docker can be used to split this between two
machines, but that distributed setup is rare

Currently supported additional options:
- Sending stats to https://beaconcha.in
- Prysm Web UI
- Grafana dashboard
- slasher - running slasher is optional and requires additional resources

Please see [Prysm Web](../Usage/WebUI.md) for Web UI support on Prysm.
