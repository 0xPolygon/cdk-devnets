# Devnets Deployment Guide

This repository provides step-by-step instructions for deploying an OP Stack rollup and connecting it to the Agglayer.

## Wallet Accounts
You will need funded wallet accounts to deploy and operate the network.

**Common**

| Role      | Variable Name     | L1 Funds Required | L2 Funds Required |
|-----------|------------------|:-----------------:|:-----------------:|
| Deployer  | DEPLOYER_ADDRESS |        ✅         |        ❌         |

**Polygon Stack**

| Role           | Variable Name         | L1 Funds Required | L2 Funds Required |
|----------------|----------------------|:-----------------:|:-----------------:|
| Aggoracle      | AGGORACLE_ADDRESS    |        ❌         |        ✅         |
| Aggsender      | AGGSENDER_ADDRESS    |        ❌         |        ❌         |
| ClaimTxManager | CLAIMTXMANAGER_ADDR  |        ❌         |        ✅         |
| Admin          | ADMIN_ADDR           |        ❌         |        ❌         |

**OP Stack**

| Role      | Variable Name            | L1 Funds Required | L2 Funds Required |
|-----------|-------------------------|:-----------------:|:-----------------:|
| Sequencer | SEQUENCER_ADDRESS       |        ❌         |        ❌         |
| Batcher   | BATCHER_ADDRESS         |        ✅         |        ❌         |


> 💡 **Tip:** The easiest way to get funds on L2 for testnet/devnet is by prefunding accounts during genesis generation. When you follow step 2 (**[Genesis Generation](02-genesis.md)**), add any accounts you want to be funded directly to the genesis file with the desired balance.





## Deployment Steps

1. **[Rollup Creation](01-rollup-creation.md)** - Submit a request to Polygon Labs and receive deployment artifacts
2. **[Genesis Generation](02-genesis.md)** - Generate the L2 genesis file with pre-deployed contracts
3. **[OP Stack Deployment](03-op-stack.md)** - Deploy L1 contracts and merge genesis allocs using `op-deployer`
4. **[Rollup Initialization](04-rollup-initialization.md)** - Initialize the rollup on-chain with required parameters
5. **[OP Succinct Configuration](05-op-succinct-config.md)** *(FEP only)* - Add and select the op-succinct configuration for L2 output proposals
6. **[Polygon Stack Common Components](06-polygon-stack-common.md)** - Aggkit configuration for any consensus type
7. **[Polygon Stack FEP Components](07-polygon-stack-fep.md)** *(FEP only)* - Run the Aggkit Prover and OP Succinct Proposer services

