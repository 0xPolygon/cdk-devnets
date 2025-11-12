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

> **Source of Truth**: For complete documentation, see the [OP Succinct Proposer Documentation](https://github.com/agglayer/op-succinct/blob/v3.3.3-agglayer/book/validity/proposer.md).

### Prerequisites

- A dedicated PostgreSQL database (must be set up separately)
- Access to L1 and L2 RPC endpoints

### Docker Image

```shell
ghcr.io/agglayer/op-succinct/op-succinct-agglayer:v3.3.3-agglayer
```

### Environment Variables

> **Source of Truth**: For complete environment variable documentation, see the [Environment Setup](https://github.com/agglayer/op-succinct/blob/v3.3.3-agglayer/book/validity/proposer.md#environment-setup) section.

### Running the Container

Make sure to include *all* of the required environment variables in the `.env` file. See the documentation for the complete list of required and optional environment variables.

```shell
docker run --rm -it \
  --env-file .env \
  ghcr.io/agglayer/op-succinct/op-succinct-agglayer:v3.3.3-agglayer
```

#### Example with Docker Compose

See the [example docker-compose.yml](https://github.com/agglayer/op-succinct/blob/v3.3.3-agglayer/docker-compose.yml) for reference.

### Additional Resources
- **GitHub**: [agglayer/op-succinct](https://github.com/agglayer/op-succinct)
