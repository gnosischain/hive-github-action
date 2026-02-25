# Client Configuration Helper

This composite action generates Hive client configuration based on input parameters. It handles both git-based and docker-based client sources, parsing client versions and images to generate the appropriate YAML configuration.

## Usage

```yaml
- uses: ./helpers/client-config
  id: client_config
  with:
    client_repos: |
      {
        "geth": "gnosischain/go-ethereum@v1.16.8-gc",
        "reth": "gnosischain/reth_gnosis@master"
      }
    client_images: |
      {
        "geth": "ghcr.io/gnosischain/geth:v1.16.8-gc",
        "reth": "ghcr.io/gnosischain/reth_gnosis:master"
      }
    client_source: 'git'
    hive_version: 'gnosischain/hive@master'
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `client_repos` | JSON object containing client versions in format `{"client": "repo@tag", ...}` | No | Default versions for all clients |
| `client_images` | JSON object containing client docker images in format `{"client": "registry:tag", ...}` | No | Default images for all clients |
| `common_client_tag` | If provided, this tag will be used for all clients, overriding individual tags/branches | No | `''` |
| `client_source` | How client images should be sourced (`git` or `docker`) | Yes | `git` |
| `hive_version` | GitHub repository and tag for hive (`repo@tag`) | No | `gnosischain/hive@master` |
| `goproxy` | Go proxy URL for Go-based clients | No | `''` |

## Outputs

### Hive Version
- `hive_repo`: Hive repository
- `hive_tag`: Hive tag

### Git Client Versions
- `geth_repo`, `geth_tag`: Go-Ethereum repository and tag
- `reth_repo`, `reth_tag`: Reth repository and tag
- `nethermind_repo`, `nethermind_tag`: Nethermind repository and tag
- `erigon_repo`, `erigon_tag`: Erigon repository and tag

### Docker Client Images
- `geth_docker_registry`, `geth_docker_tag`: Go-Ethereum docker registry and tag
- `reth_docker_registry`, `reth_docker_tag`: Reth docker registry and tag
- `nethermind_docker_registry`, `nethermind_docker_tag`: Nethermind docker registry and tag
- `erigon_docker_registry`, `erigon_docker_tag`: Erigon docker registry and tag

### Final Configuration
- `client_config`: YAML client configuration for Hive

## Logic

1. **Parsing**: The action first parses the input JSON objects for client versions and images
2. **Tag Override**: If `common_client_tag` is provided, it overrides individual client tags
3. **Configuration Generation**: Based on `client_source`, it generates either git-based or docker-based client configurations
4. **Output**: Returns both the parsed individual components and the final YAML configuration

## Supported Clients

- `geth` (Go-Ethereum) — [gnosischain/go-ethereum](https://github.com/gnosischain/go-ethereum)
- `reth` (Reth) — [gnosischain/reth_gnosis](https://github.com/gnosischain/reth_gnosis)
- `nethermind` (Nethermind)
- `erigon` (Erigon)
