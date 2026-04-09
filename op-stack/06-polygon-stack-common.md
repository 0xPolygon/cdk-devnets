# Polygon Stack Common Deployment

This document describes the common components required for Polygon Stack deployment that work with any type of consensus network.

## Aggkit

### Prerequisites

- Access to L1 and L2 RPC endpoints

### Environment Variables

```shell
# See Component Versions table in README.md for current values
export aggkit_version="<aggkit_version>"
```

### Configuration

> **Source of Truth**: For complete configuration documentation, see the [Aggkit Documentation](https://github.com/agglayer/aggkit/blob/develop/docs/SUMMARY.md).

Aggkit uses a TOML configuration file. Create a configuration file at `/etc/aggkit/config.toml`. See the [example configuration file](https://github.com/agglayer/aggkit/blob/develop/config.toml.example) for reference.

### Running the Container

Run Aggkit using Docker with the configuration file mounted. The command specifies which components to run:

```shell
docker run --rm -it \
  -v /path/to/config.toml:/etc/aggkit/config.toml \
  -v /path/to/data:/data \
  ghcr.io/agglayer/aggkit:${aggkit_version} \
  aggkit run --cfg=/etc/aggkit/config.toml --components=aggoracle,aggsender
```

**Components:**
- `aggoracle`: AggOracle service for managing global exit roots
- `aggsender`: AggSender service for sending certificates

You can run individual components or any combination by adjusting the `--components` flag.

### Additional Resources
- **GitHub**: [agglayer/aggkit](https://github.com/agglayer/aggkit)
