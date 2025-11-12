# Genesis File Generation

This document describes how to generate the L2 genesis file used by the rollup creation flow. The recommended approach is to merge OP Stack and Polygon genesis files with pre-deployed contracts, ensuring pre-deployed addresses and balances are present in the L2 genesis.

## TL;DR

- **Recommended**: Merge OP Stack + Polygon genesis with pre-deployed contracts
- **Alternative**: Manual L2 contract deployment (advanced; outside the scope of this guide)

1. Checkout the agglayer-contracts repository and install dependencies
2. Create and configure the parameter file using values from `combined.json`
3. Download the allocs file from cdk-contracts-tooling (Bali, Cardona, etc) as `genesis-base.json`
4. Run the Hardhat script to generate genesis files
5. Rename the output files to canonical names

## Prerequisites

- Node.js (for Hardhat) and `npm` installed
- Dependencies installed as described in the repository README
- Familiarity with `jq`, `gzip`, and `base64` (optional, for merging allocs)

## Step 1: Checkout Repository and Install

Clone the repository version that matches this guide (example uses v12.1.6):

```shell
git clone --depth 1 --branch v12.1.6 https://github.com/agglayer/agglayer-contracts
cd agglayer-contracts
npm install
```

## Step 2: Create Parameter File

Copy the example parameter file:

```shell
cp ./tools/createSovereignGenesis/create-genesis-sovereign-params.json.example ./tools/createSovereignGenesis/create-genesis-sovereign-params.json
```

Update the file with relevant information from your `combined.json`. Example configuration:

```json
{
    "rollupManagerAddress": "0xe983fD1798689eee00c0Fb77e79B8f372DF41060",
    "rollupID": 3,
    "chainID": 1001,
    "gasTokenAddress": "0x0000000000000000000000000000000000000000",
    "bridgeManager": "0x8576158a89648aA88B6036f47B8b74Fc0C2b5c66",
    "sovereignWETHAddress": "0x0000000000000000000000000000000000000000",
    "sovereignWETHAddressIsNotMintable": false,
    "globalExitRootUpdater": "",
    "globalExitRootRemover": "0x8576158a89648aA88B6036f47B8b74Fc0C2b5c66",
    "emergencyBridgePauser": "0x8576158a89648aA88B6036f47B8b74Fc0C2b5c66",
    "emergencyBridgeUnpauser": "0x8576158a89648aA88B6036f47B8b74Fc0C2b5c66",
    "proxiedTokensManager": "0xB55B27Cca633A73108893985350bc26B8A00C43a",
    "setPreMintAccounts": true,
    "preMintAccounts": [
        {
            "balance": "1000000000000000000",
            "address": "0x8576158a89648aA88B6036f47B8b74Fc0C2b5c66"
        },
        {
            "balance": "1000000000000000000",
            "address": "0xb420EAAbFeFA05b39dE520f811325A463E023954"
        }
    ],
    "setTimelockParameters": true,
    "timelockParameters": {
        "adminAddress": "0x8576158a89648aA88B6036f47B8b74Fc0C2b5c66",
        "minDelay": 3600
    },
    "useAggOracleCommittee": true,
    "aggOracleCommittee": [
        "0x8576158a89648aA88B6036f47B8b74Fc0C2b5c66",
        "0xb420EAAbFeFA05b39dE520f811325A463E023954"
    ],
    "quorum": 1,
    "aggOracleOwner": "0x8576158a89648aA88B6036f47B8b74Fc0C2b5c66",
    "formatGenesis": "geth"
}
```

## Step 3: Download Allocs File

Download the allocs file from the [cdk-contracts-tooling](https://github.com/0xPolygon/cdk-contracts-tooling) repository based on your environment:

- **Bali**: [allocs.json](https://raw.githubusercontent.com/0xPolygon/cdk-contracts-tooling/main/genesis/pp/bali/allocs.json)
- **Cardona**: [allocs.json](https://raw.githubusercontent.com/0xPolygon/cdk-contracts-tooling/main/genesis/pp/cardona/allocs.json)
- **Mainnet**: [allocs.json](https://raw.githubusercontent.com/0xPolygon/cdk-contracts-tooling/main/genesis/pp/mainnet/allocs.json)

Download the appropriate file and save it as `genesis-base.json` in the `./tools/createSovereignGenesis/` directory:

```shell
# For Bali (example)
curl -o ./tools/createSovereignGenesis/genesis-base.json \
  https://raw.githubusercontent.com/0xPolygon/cdk-contracts-tooling/main/genesis/pp/bali/allocs.json

# For Cardona
# curl -o ./tools/createSovereignGenesis/genesis-base.json \
#   https://raw.githubusercontent.com/0xPolygon/cdk-contracts-tooling/main/genesis/pp/cardona/allocs.json
```

## Step 4: Generate Genesis Files

Run the Hardhat script to assemble the sovereign genesis using your parameter file and the downloaded allocs file:

```shell
npx hardhat run ./tools/createSovereignGenesis/create-sovereign-genesis.ts --network sepolia
```

The script produces `genesis-rollupID-*.json` and `output-rollupID-*.json` files under `./tools/createSovereignGenesis/`.

## Step 5: Rename Outputs

Rename the generated artifacts to the canonical names used by other scripts in this repository:

```shell
mv ./tools/createSovereignGenesis/genesis-rollupID-*.json ./tools/createSovereignGenesis/polygon-genesis.json
mv ./tools/createSovereignGenesis/output-rollupID-*.json ./tools/createSovereignGenesis/polygon-genesis-info.json
```
