# Network Deployment (OP Stack)

This document explains how to deploy an L2 using the OP Stack's `op-deployer` tool. It follows the official Optimism deployment flow and focuses on the steps most relevant to this repository.

## References

- [Optimism L2 Rollup Tutorial](https://docs.optimism.io/operators/chain-operators/tutorials/create-l2-rollup)
- [op-deployer Tool Documentation](https://docs.optimism.io/operators/chain-operators/tools/op-deployer)

> **Note**: All commands assume you're running from the repository root.

## TL;DR

1. Initialize an `op-deployer` workdir
2. Edit `intent.toml` with deployment parameters
3. Deploy L1 contracts using `op-deployer apply`
4. Merge existing genesis allocs (e.g., Polygon) into the op-deployer state
5. Generate final `genesis.json` and `rollup.json` files via `op-deployer inspect`

### Environment Variables

Set the following environment variables before proceeding:

```shell
export l1_chain_id=11155111
export l2_chain_id=0
export l1_rpc_url="https://<your-l1-rpc>"
export deployer_private_key=0x...
export op_deployer_version="v0.4.5"
```

## Step 1: Initialize the Deployer Workdir

Create a local `deployer` folder and initialize the op-deployer state:

```shell
docker run --rm -v "$(pwd)/deployer:/deployer" -it \
	us-docker.pkg.dev/oplabs-tools-artifacts/images/op-deployer:${op_deployer_version} \
	/usr/local/bin/op-deployer init \
		--l1-chain-id ${l1_chain_id} \
		--l2-chain-ids ${l2_chain_id} \
		--workdir /deployer
```

Edit the generated `deployer/intent.toml` with your deployment parameters.

## Step 2: Deploy L1 Contracts

When your `intent.toml` is ready, deploy the L1 contracts required by the OP Stack:

```shell
docker run --rm -v "$(pwd)/deployer:/deployer" -it \
	us-docker.pkg.dev/oplabs-tools-artifacts/images/op-deployer:${op_deployer_version} \
	/usr/local/bin/op-deployer apply \
		--workdir /deployer \
		--l1-rpc-url ${l1_rpc_url} \
		--private-key ${deployer_private_key}
```

This writes the deployer state to `deployer/state.json` and related artifacts.

## Step 3: Merge OP + Polygon Genesis with Pre-deployed Contracts

This step is required when you have pre-deployed contracts or existing chain allocs (for example, from a `polygon-genesis.json`). You must merge those allocs into the op-deployer state so the final L2 genesis includes the pre-deployed addresses and balances.

The commands below:
1. Extract the base64/gzip-encoded allocs from the op-deployer state
2. Merge them with your Polygon alloc fragment
3. Write the merged allocs back into the state

```shell
# Extract the allocs
cat deployer/state.json | jq -r '.opChainDeployments[].allocs' | base64 -d | gzip -d > allocs.json

# Merge
jq -s add allocs.json files/polygon-genesis.json | gzip | base64 > merge

# Create a copy of the original state
cp deployer/state.json deployer/original-state.json

# Replace the original allocs by the merged
cat deployer/state.json | jq ".opChainDeployments[].allocs=\"$( cat merge )\"" > state.json && mv state.json deployer/state.json

# Cleanup
rm allocs.json merge
```

## Step 4: Generate Final Artifacts

Once the state is ready, use `op-deployer inspect` to produce the final `genesis.json` and `rollup.json` for the L2:

```shell
docker run --rm -v "$(pwd)/deployer:/deployer" -it \
	us-docker.pkg.dev/oplabs-tools-artifacts/images/op-deployer:${op_deployer_version} \
	/usr/local/bin/op-deployer inspect genesis --workdir /deployer ${l2_chain_id} > ./deployer/genesis.json

docker run --rm -v "$(pwd)/deployer:/deployer" -it \
	us-docker.pkg.dev/oplabs-tools-artifacts/images/op-deployer:${op_deployer_version} \
	/usr/local/bin/op-deployer inspect rollup --workdir /deployer ${l2_chain_id} > ./deployer/rollup.json
```

These files serve as inputs for running the OP Stack nodes or for further tooling in this repository.
