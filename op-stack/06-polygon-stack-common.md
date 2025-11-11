# Polygon Stack Common Deployment

This document describes the common components required for Polygon Stack deployment that work with any type of consensus network.

## Aggkit

### Prerequisites

- Access to L1 and L2 RPC endpoints
- Contract addresses for Polygon Rollup Manager, Global Exit Root Manager, and Bridge
- Private keys for aggsender and aggoracle
- Genesis file and rollup creation information

### Docker Image

```shell
ghcr.io/agglayer/aggkit:0.7.1
```

### Configuration

Aggkit uses a TOML configuration file. Create a configuration file at `/etc/aggkit/config.toml`. Example configuration:

```toml
NetworkID = 0

PathRWData = "/data"

L1URL = "https://<your-l1-rpc>"
L2URL = "http://<your-l2-rpc>"

polygonBridgeAddr = "0x..."

rollupCreationBlockNumber = 0
rollupManagerCreationBlockNumber = 0
genesisBlockNumber = 0

[L1Config]
chainId = 11155111
polygonZkEVMGlobalExitRootAddress = "0x..."
polygonRollupManagerAddress = "0x..."
polTokenAddress = "0x..."
polygonZkEVMAddress = "0x..."

[L2Config]
GlobalExitRootAddr = "0x..."

[AggSender]
AggsenderPrivateKey = {Method = "GCP", KeyName = "path/to/aggsender/kms/key"}
Mode = "AggchainProof"
RequireNoFEPBlockGap = true

[AggSender.OptimisticModeConfig]
TrustedSequencerKey = {Method = "GCP", KeyName = "path/to/aggsender/kms/key"}
OpNodeURL = "http://<your-op-node-rpc>"
RequireKeyMatchTrustedSequencer = true

[AggSender.AggkitProverClient]
URL = "http://<aggkit-prover>:4446"
UseTLS = "false"

[AggSender.AgglayerClient.GRPC]
URL = "<agglayer-grpc-endpoint>:443"
UseTLS = "true"

[AggOracle]
EnableAggOracleCommittee = true

[AggOracle.EVMSender]
AggOracleCommitteeAddr = "0x..."

[AggOracle.EVMSender.EthTxManager]
PrivateKeys = [{Method = "GCP", KeyName = "path/to/aggoracle/kms/key"}]

[AggOracle.EVMSender.EthTxManager.Etherman]
# This field should be populated with L2ChainID
L1ChainID = 888
```

**Key configuration fields:**
- **NetworkID**: Network identifier for the rollup
- **L1URL/L2URL**: RPC endpoints for L1 and L2 networks
- **Contract addresses**: Polygon Rollup Manager, Global Exit Root Manager, Bridge, and token addresses
- **Block numbers**: Rollup creation, manager creation, and genesis block numbers
- **AggSender**: Configuration for sending aggregated certificates, including prover client and Agglayer client settings
- **AggOracle**: Configuration for the oracle service, including committee settings and private key management

> **Note**: Private keys can be configured using GCP KMS (as shown) or file-based keystores. Adjust the `Method` and `KeyName`/`Path` accordingly.

### Running the Container

Run Aggkit using Docker with the configuration file mounted. The command specifies which components to run:

```shell
docker run --rm -it \
  -v /path/to/config.toml:/etc/aggkit/config.toml \
  -v /path/to/keystores:/etc/aggkit \
  -v /path/to/data:/data \
  ghcr.io/agglayer/aggkit:0.7.1 \
  aggkit run --cfg=/etc/aggkit/config.toml --components=aggoracle,aggsender,bridge
```

**Components:**
- `aggoracle`: AggOracle service for managing global exit roots
- `aggsender`: AggSender service for sending certificates
- `bridge`: Bridge service for L1/L2 synchronization

You can run individual components or any combination by adjusting the `--components` flag.

### Additional Resources

- **GitHub**: [agglayer/aggkit](https://github.com/agglayer/aggkit)
