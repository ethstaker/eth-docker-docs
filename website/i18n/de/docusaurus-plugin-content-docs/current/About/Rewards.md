---
title: Chain Rewards	 
sidebar_position: 4
sidebar_label: Chain Rewards
---

Validators that participate in securing the beacon chain and execute "duties" get rewarded for this by new issuance of ETH. In addition, validators receive priority fees paid by users, and optionally MEV, Maximal Extractable Value.

1) Validator rewards

Validators have three duties: Attestations, once every epoch; block production, once every two months or so; sync committees, once every two-and-a-half years or so, on average. Block production and sync committees are randomly assigned, and the time between them therefore varies widely for any one validator. 

They are rewarded for executing those duties by new ETH issuance to the "validator balance". The current APY can be seen on the [official launchpad](https://launchpad.ethereum.org/).

If the validator is down and not executing its duties, it will be penalized at a rate slightly lower than the rewards for the same period of time.

Rewards are withdrawn automatically to the [withdrawal address](https://ethereum.org/en/staking/withdrawals/).

2) Priority fees

Users of Ethereum set a [priority fee](https://ethereum.org/en/developers/docs/gas/#priority-fee) for their transactions. This fee is paid to block proposers and is immediately liquid. In order for that fee to be paid, a `--suggested-fee-recipient` needs to be configured. This is why Eth Docker prompts for an Ethereum address to send priority fees to.

This address could be a hardware wallet, a software wallet, or even a multi-sig contract.

3) MEV

[MEV](https://ethereum.org/en/developers/docs/mev/), or "maximal extractable value", is a controversial topic. Node operators can extract MEV by accepting blocks built by "searchers", via a small side program called ["mev-boost"](https://ethresear.ch/t/mev-boost-merge-ready-flashbots-architecture/11177) by Flashbots. In this case, the CL ... Consensus Layer client such as Nimbus, Teku, &c ... will, when asked to procure a block to propose, get blocks from MEV relays via mev-boost and from the EL ... Execution Layer client such as Besu, Geth, &c ... and then choose whichever block from the relay pays best. The EL does not currently communicate its expected payout and would only be chosen when the relay offers no block. For this, the relay has to be trusted to deliver valid blocks.

Rewards from MEV are paid to the same `--suggested-fee-recipient` address that priority fees go to.

The `-min-bid` flag allows setting a minimum bid value for MEV blocks. You can find this flag in `.env` file as `MEV_MIN_BID` (e.g. 0,.05). If left empty, no minimum bid is set. This means that if the relay bids offer less than 0.05 ETH in rewards, MEV-Boost will not choose those bids and will instead produce blocks locally. Overall, this setting will help to decentralize block building, as it will make it less likely that an external block builder is chosen when the profit is low. 
