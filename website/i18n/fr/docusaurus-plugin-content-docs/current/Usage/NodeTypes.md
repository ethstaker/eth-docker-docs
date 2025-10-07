---
title: Archive, Full or Expired node
sidebar_position: 2
sidebar_label: History and Pruning
---

## Types of nodes

You can run an Archive node, with all history and full lookup for all historical transactions. This may take large amounts of space, depending on the execution layer client.

You can run a Full node, with all history and limited lookup for historical transactions. This fits into a 4TB drive and is often the choice for RPC nodes.

You can run an Expired node, with pre-merge history and receipts gone. This fits into a 2TB drive and is foten the choice for validator nodes.

This is controlled by variables in `.env`, which can be set with `nano .env`. Switching from one type to another often requires a full resync.

`CL_ARCHIVE_NODE` - run the consensus layer node as an archive, including blobs where supported  
`CL_MINIMAL_NODE` - run the consensus layer node with minimal storage

`EL_ARCHIVE_NODE` - run the execution layer node as an archive. The required space can vary widely depending on the client: From right around 2 TB to well over 50TB  
`EL_MINIMAL_NODE` - run the execution layer node with history expiry. `true` is pre-merge history expiry; `rolling` is 1-year rolling if the client supports it;
`aggressive` expires all but the last few blocks, if the client supoorts it

## Switch from Full to Expired node

You can use `./ethd prune-history` to switch the client to history expiry, in some cases without resync. Note that Besu requires 200 GiB free for this, and
it will take less space if instead it is resynced with `./ethd resync-execution` while `EL_MINIMAL_NODE=true`

## State pruning

In addition to history, execution layer clients also carry state. Historically, the size of the state DB has been growing and can be pruned periodically.
Today, only Nethermind still requires a state prune. Note this is entirely separate from history expiry. It applies to Full and Expired nodes. Archive
nodes should never have their state pruned.

### Automatic Nethermind prune

By default, Nethermind will prune when free disk space falls below 350 GiB on mainnet, or 50 GiB on testnet. If you
want to disable that, `nano .env` and change `AUTOPRUNE_NM` to `false`.

If you have disabled automatic prune, you can run `./ethd prune-nethermind`. It will check prerequisites, online prune Nethermind, and restart it.

### Continuous Besu prune

Besu continuously prunes with BONSAI, and from 24.1.0 on also prunes its trie-logs. A long-running Besu may benefit
from a manual trie-log prune, once.

If you have a large amount of trie logs, run `./ethd prune-besu` on a  long-running Besu. It will check prerequisites, offline prune Besu trie-logs, and
restart it.

### Continuous Geth prune

Geth continuously prunes if synced with PBSS. If you are using an old hash-synced Geth, run `./ethd resync-execution`
to use PBSS. This will cause downtime while Geth syncs, which can take 6-12 hours.
