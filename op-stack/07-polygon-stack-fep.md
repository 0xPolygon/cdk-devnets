# Polygon Stack FEP Deployment

This document describes how to run the components required for Polygon Stack FEP (Full Execution Proof) deployment.

## Aggkit Prover

### Prerequisites

- Access to L1 and L2 RPC endpoints

### Docker Image

```shell
ghcr.io/agglayer/aggkit-prover:1.7.1
```

### Configuration

The Aggkit Prover uses a TOML configuration file. Create a minimal configuration file at `/etc/aggkit/aggkit-prover-config.toml`:

```toml
grpc-endpoint = "0.0.0.0:4446"

[log]
level = "debug"
outputs = []
format = "json"

[telemetry]
prometheus-addr = "0.0.0.0:9090"

[shutdown]
runtime-timeout = "5s"

[aggchain-proof-service.aggchain-proof-builder]
network-id = 0
proving-timeout = "1h"

[aggchain-proof-service.aggchain-proof-builder.primary-prover.network-prover]
proving-timeout = "1h"

[aggchain-proof-service.aggchain-proof-builder.contracts]
l1-rpc-endpoint = "https://<your-l1-rpc>"
l2-execution-layer-rpc-endpoint = "http://<your-l2-execution-rpc>"
l2-consensus-layer-rpc-endpoint = "http://<your-l2-consensus-rpc>"
polygon-rollup-manager = "0x..."
global-exit-root-manager-v2-sovereign-chain = "0x..."
evm-sketch-genesis = "/etc/aggkit/genesis.json"

[aggchain-proof-service.proposer-service]
l1-rpc-endpoint = "https://<your-l1-rpc>"
mock = false

[aggchain-proof-service.proposer-service.client]
proposer-endpoint = "http://<op-succinct-proposer>:50001"
sp1-cluster-endpoint = "https://<sp1-cluster-endpoint>"
request-timeout = 3600
proving-timeout = 3600

[primary-prover.network-prover]
proving-timeout = "1h"
```

### Running the Container

Run the Aggkit Prover using Docker with the configuration file mounted:

```shell
docker run --rm -it \
  -v /path/to/aggkit-prover-config.toml:/etc/aggkit/aggkit-prover-config.toml \
  -v /path/to/op-stack/genesis.json:/etc/aggkit/genesis.json \
  ghcr.io/agglayer/aggkit-prover:1.7.1 \
  /usr/local/bin/aggkit-prover run --config-path /etc/aggkit/aggkit-prover-config.toml
```

### Additional Resources

- **GitHub**: [agglayer/provers](https://github.com/agglayer/provers)

## OP Succinct Proposer

### Prerequisites

- A dedicated PostgreSQL database (must be set up separately)
- Access to L1 and L2 RPC endpoints

### Docker Image

```shell
ghcr.io/agglayer/op-succinct/op-succinct-agglayer:v3.3.3-agglayer
```

### Environment Variables

Set the following environment variables before running the container:

```shell
# Database configuration
export DATABASE_URL="postgresql://user:password@host:port/database"
export DB_PATH=""                              # Alternative database path
export USE_CACHED_DB=""                        # Enable cached database

# RPC endpoints
export L1_RPC="https://<your-l1-rpc>"
export L1_BEACON_RPC="https://<your-l1-beacon-rpc>"
export L2_NODE_RPC="http://<your-l2-node>"
export L2_RPC="http://<your-l2-rpc>"
export NETWORK_RPC_URL=""                      # Network RPC URL

# Contract addresses
export L2OO_ADDRESS="0x..."                    # L2 Output Oracle contract address
export PROVER_ADDRESS="0x..."                  # Prover contract address

# Configuration
export OP_SUCCINCT_CONFIG_NAME=""              # OP Succinct configuration name
export OP_SUCCINCT_MOCK=""                     # Enable mock mode (testing)

# Performance tuning
export MAX_CONCURRENT_PROOF_REQUESTS=""        # Maximum concurrent proof requests
export MAX_CONCURRENT_WITNESS_GEN=""           # Maximum concurrent witness generation
export WITNESS_GEN_TIMEOUT=""                  # Witness generation timeout
export POLL_INTERVAL=""                        # Polling interval
export RANGE_PROOF_INTERVAL=""                 # Range proof interval

# AggLayer specific
export AGGLAYER=""                             # AggLayer configuration
export AGG_PROOF_MODE=""                       # Aggregation proof mode

# Monitoring
export METRICS_ENABLED=""                      # Enable metrics
export METRICS_PORT=""                         # Metrics port

# gRPC
export GRPC_ADDRESS=""                         # gRPC server address

# Logging
export RUST_LOG=""                             # Rust log level
```

### Running the Container

Run the OP Succinct Proposer using Docker. All environment variables must be passed to the container:

```shell
docker run --rm -it \
  --env DATABASE_URL="${DATABASE_URL}" \
  --env DB_PATH="${DB_PATH}" \
  --env USE_CACHED_DB="${USE_CACHED_DB}" \
  --env L1_RPC="${L1_RPC}" \
  --env L1_BEACON_RPC="${L1_BEACON_RPC}" \
  --env L2_NODE_RPC="${L2_NODE_RPC}" \
  --env L2_RPC="${L2_RPC}" \
  --env NETWORK_RPC_URL="${NETWORK_RPC_URL}" \
  --env L2OO_ADDRESS="${L2OO_ADDRESS}" \
  --env PROVER_ADDRESS="${PROVER_ADDRESS}" \
  --env OP_SUCCINCT_CONFIG_NAME="${OP_SUCCINCT_CONFIG_NAME}" \
  --env OP_SUCCINCT_MOCK="${OP_SUCCINCT_MOCK}" \
  --env MAX_CONCURRENT_PROOF_REQUESTS="${MAX_CONCURRENT_PROOF_REQUESTS}" \
  --env MAX_CONCURRENT_WITNESS_GEN="${MAX_CONCURRENT_WITNESS_GEN}" \
  --env WITNESS_GEN_TIMEOUT="${WITNESS_GEN_TIMEOUT}" \
  --env POLL_INTERVAL="${POLL_INTERVAL}" \
  --env RANGE_PROOF_INTERVAL="${RANGE_PROOF_INTERVAL}" \
  --env AGGLAYER="${AGGLAYER}" \
  --env AGG_PROOF_MODE="${AGG_PROOF_MODE}" \
  --env METRICS_ENABLED="${METRICS_ENABLED}" \
  --env METRICS_PORT="${METRICS_PORT}" \
  --env GRPC_ADDRESS="${GRPC_ADDRESS}" \
  --env RUST_LOG="${RUST_LOG}" \
  ghcr.io/agglayer/op-succinct/op-succinct-agglayer:v3.3.3-agglayer
```

> **Note**: Only set environment variables that are required for your configuration. Optional variables can be omitted or left empty.

### Additional Resources

- **GitHub**: [agglayer/op-succinct](https://github.com/agglayer/op-succinct)
