---
title: "Aztec Sequencer"
sidebar_position: 7
sidebar_label: Aztec Sequencer
---

This guide covers enabling and configuring an Aztec Sequencer node within the `eth-docker` stack.

## Prerequisites

1. An active `eth-docker` stack with `execution`, `consensus`, and `web3signer` running.
2. At least **200,000 AZTEC tokens** staked for the sequencer identity.
3. **≥ 0.1 ETH** funded in the publisher account(s) for L1 gas.

## 1. Enable the Services

Add `web3signer.yml` and `aztec-sequencer.yml` to your `.env` file:

```env
COMPOSE_FILE=...existing files...,web3signer.yml,aztec-sequencer.yml
```

## 2. Configure Keys (Step-by-Step Key Migration)

Because Aztec requires a hybrid keystore approach (Web3Signer for ETH, inline for BLS), operators must manually migrate the ETH key into Web3Signer.

### 2.1 Generate Aztec Keystore

Use the Aztec CLI on a secure machine to generate your validator keys:

```bash
aztec validator-keys new \
  --fee-recipient 0x0000000000000000000000000000000000000000000000000000000000000000 \
  --staker-output \
  --count 1
```

This creates `key1.json` (private) in `~/.aztec/keystore/`.

### 2.2 Import ETH Key to Web3Signer

From `key1.json`, extract the 66-character hex string (starting with `0x`) from `attester.eth` (and `publisher` if used). Strip the `0x` prefix to get the 64-character raw private key.

Add this key to Web3Signer by creating a file named `validator.key` (the `.key` extension is required) containing *only* the 64-character hex string. Place this file in your `eth-docker` Web3Signer keys volume (typically `~/.eth/web3signer/keys/`). Restart the `web3signer` container to load it. *(Alternatively, convert the key to an EIP-2335 JSON V3 keystore using `ethdo` for production environments).*

### 2.3 Configure the Aztec Keystore

Copy the generated private keystore to your `eth-docker` directory:

```bash
cp ~/.aztec/keystore/key1.json aztec/keys/keystore.json
```

Edit `aztec/keys/keystore.json`:
- Replace the 66-character `attester.eth` private key with the **42-character Ethereum address** derived from the key you just imported into Web3Signer.
- Do the same for the `publisher` address if applicable.
- **Crucial:** Leave the `bls` key exactly as it is (the 66-character raw private key). Aztec handles BLS signing internally and does not route BLS keys to Web3Signer.

### 2.4 Verify Permissions

Ensure the file is readable by the Docker user (UID 10000 by default for Aztec, but mounted read-only so standard permissions apply).

## 3. Update Environment Variables

In your `eth-docker` `.env` file, set the following. You can find your public IP by running `curl -s ifconfig.me`:

```env
AZTEC_P2P_IP=<YOUR_PUBLIC_IP> # Required for reliable P2P discovery
LOG_LEVEL=info
```

## 4. Deploy and Verify

```bash
./ethd up
```

Verify the container is healthy and metrics are being scraped:

```bash
./ethd ps aztec-sequencer
curl http://localhost:8080/metrics # Should return Prometheus metrics
```

Metrics and logs will automatically be collected by Grafana Alloy (if enabled via `grafana.yml` or `grafana-cloud.yml`) and forwarded to your configured Grafana Cloud or local stack.