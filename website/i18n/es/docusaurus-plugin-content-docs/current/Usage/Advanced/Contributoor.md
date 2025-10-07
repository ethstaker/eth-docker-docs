---
title: EthPandaOps Contributoor.
sidebar_position: 9
sidebar_label: EthPandaOps Contributoor
---

Contributoor allows node operators to securely share metrics with the ethPandaOps community, helping build a comprehensive view of network health. By contributing your node metrics, you help improve understanding of the Ethereum network and assist in identifying issues.

More information: [Contributoor: A Lightweight Beacon Node Companion](https://ethpandaops.io/posts/contributoor-beacon-node-companion/)

## Ready to Contribute?

Who Can Join: For now, individual home stakers running their own beacon nodes  

Getting Started: Some basic details are needed to get you started - [Apply here](https://forms.gle/S7g5g8nB8aGG8aTX6)  

Get in Touch: Join their [Telegram](https://t.me/+JanoQFu_nO8yNzQ1) group or find them on [Twitter](https://x.com/ethpandaops)  

## Setup

Note that configuring contributoor requires that you have obtained username and password following the procedure above.

- Run `./ethd update`  
- Then edit your .env file:  
-- add :contributoor.yml to the end of COMPOSE_FILE variable  
-- set CONTRIBUTOOR_USERNAME variable to your contributoor username  
-- set CONTRIBUTOOR_PASSWORD variable to your contributoor password  
- Run `./ethd update`  
- Run `./ethd up`  

## Check logs

`./ethd logs -f contributoor` should show it starting up and begin collecting events.
