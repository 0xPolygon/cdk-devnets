# Polygon Stack Common Deployment

This document describes the common components required for Polygon Stack deployment that work with any type of consensus network.

## Aggkit

### Prerequisites

- Access to L1 and L2 RPC endpoints

### Docker Image

```shell
ghcr.io/agglayer/aggkit:0.7.1
```

### Configuration

> **Source of Truth**: For complete configuration documentation, see the [Aggkit Documentation](https://github.com/agglayer/aggkit/blob/v0.7.1/docs/SUMMARY.md).

Aggkit uses a TOML configuration file. Create a configuration file at `/etc/aggkit/config.toml`. See the [example configuration file](https://github.com/agglayer/aggkit/blob/v0.7.1/config.toml.example) for reference.

### Running the Container

Run Aggkit using Docker with the configuration file mounted. The command specifies which components to run:

```shell
docker run --rm -it \
  -v /path/to/config.toml:/etc/aggkit/config.toml \
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
