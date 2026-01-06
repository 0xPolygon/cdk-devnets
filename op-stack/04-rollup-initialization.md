# Rollup Initialization

This document explains how to generate rollup initialization artifacts and run the rollup initialization script for an Agglayer rollup.

## References

- [Initialize Rollup README](https://github.com/agglayer/agglayer-contracts/blob/v12.1.6/tools/initializeRollup/README.md) - Full usage details, configuration options, and examples

## TL;DR

1. Export required environment variables (RPC endpoints, starting block, op-succinct image tag)
2. Generate `aggchainParams` (L2 output at a block) and save to a working directory
3. Use the op-succinct image to fetch the L2 output-oracle configuration
4. Fill the initialization JSON and run the Hardhat initialization script

## Step 1: Setup Environment Variables

Export the following variables (replace placeholders with values for your environment):

```shell
export l1_rpc_url="https://<your-l1-rpc>"
export l2_node_url="https://<your-l2-node>"
export l2_rpc_url="http://<your-l2-rpc>" 
export starting_block_number=1
export op_succinct_version="v3.4.0-rc.1-agglayer"
```

> **Tip**: Keep these values private and do not commit them to version control.

## Step 2: Generate aggchainParams

Generate the L2 output at the starting block and write it to `output.json`:

```shell
cast rpc --rpc-url "$l2_node_url" optimism_outputAtBlock $(printf "0x%x" $starting_block_number) | jq '.' > output.json
```

This `output.json` file can be used to help construct `aggchainParams`.

## Step 3: Create Initialization Working Directory

Create a directory for the initialization run and write a minimal `.env` file consumed by the op-succinct helper:

```shell
mkdir -p initialize_${op_succinct_version}
cd initialize_${op_succinct_version}

cat > .env <<EOF
L1_RPC="${l1_rpc_url}"
L1_BEACON_RPC="${l1_rpc_url}"
L2_NODE_RPC="${l2_node_url}"
L2_RPC="${l2_rpc_url}"
STARTING_BLOCK_NUMBER="${starting_block_number}"
EOF
```

## Step 4: Fetch L2 Output-Oracle Configuration

Run the op-succinct container to generate/fetch the L2 output-oracle configuration file (`opsuccinctl2ooconfig.json`) into the working directory:

```shell
docker run --rm -it \
    --env OP_SUCCINCT_L2_OUTPUT_ORACLE_CONFIG_PATH=/tmp/env/opsuccinctl2ooconfig.json \
    --platform linux/amd64 \
    -v "$(pwd)":/tmp/env \
    ghcr.io/agglayer/op-succinct/op-succinct-agglayer:${op_succinct_version} \
    /bin/bash -c "fetch-l2oo-config --env-file /tmp/env/.env"
```

After completion, you should have `opsuccinctl2ooconfig.json` (or other output files) in the current directory.

## Step 5: Prepare Rollup Initialization JSON

Copy the example initialization JSON file:

```shell
cp ./tools/initializeRollup/initialize_rollup.json.example ./tools/initializeRollup/initialize_rollup.json
```

Edit the file with your values. Example configuration:

```json
{
    "type": "EOA",
    "trustedSequencerURL": "http://<your-l1-rpc>",
    "networkName": "zkevm",
    "trustedSequencer": "<AGGSENDER_ADDRESS>",
    "chainID": 0,
    "rollupAdminAddress": "<ADMIN_ADDRESS>",
    "consensusContractName": "AggchainFEP",
    "gasTokenAddress": "0x0000000000000000000000000000000000000000",
    "deployerPvtKey": "",
    "maxFeePerGas": "",
    "maxPriorityFeePerGas": "",
    "multiplierGas": "",
    "timelockDelay": 0,
    "timelockSalt": "",
    "rollupManagerAddress": "0x0000000000000000000000000000000000000000",
    "aggchainParams": {
        "initParams": {
            "l2BlockTime": <l2BlockTime-from-opsuccinctl2ooconfig.json>,
            "rollupConfigHash": "<rollupConfigHash-from-opsuccinctl2ooconfig.json>",
            "startingOutputRoot": "<startingOutputRoot-from-opsuccinctl2ooconfig.json>",
            "startingBlockNumber": <startingBlockNumber-from-opsuccinctl2ooconfig.json>,
            "startingTimestamp": 7000000,
            "submissionInterval": 1,
            "optimisticModeManager": "<ADMIN_ADDRESS>",
            "aggregationVkey": "<aggregationVkey-from-opsuccinctl2ooconfig.json>",
            "rangeVkeyCommitment": "<rangeVkeyCommitment-from-opsuccinctl2ooconfig.json>"
        },
        "useDefaultVkeys": true,
        "useDefaultSigners": false,
        "signers": [
            {
                "addr": "<AGGSENDER_ADDRESS>",
                "url": "https://<your-aggsender-rpc>"
            }
        ],
        "threshold": 1,
        "vKeyManager": "<ADMIN_ADDRESS>"
    }
}
```

## Step 6: Run the Initialization Script

Install dependencies and run the Hardhat initialization script (example uses `sepolia` network):

```shell
npm install
npx hardhat run ./tools/initializeRollup/initializeRollup.ts --network sepolia
```
