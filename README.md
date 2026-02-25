# Gnosis Hive 🐝 Github Action

This action is a wrapper around [Gnosis Hive](https://github.com/gnosischain/hive). It allows you to run tests with different clients and simulators. It also supports uploading test results to GCS (Google Cloud Storage) and/or as a workflow artifact.

> ⚠️ **Note:** This action is still under development and may introduce breaking changes. If you want to use it in your workflows, make sure to reference to a specific commit hash or tag/release.

## Inputs

### Test Configuration

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `simulator` | Simulator to run (e.g. gnosis/sync) | Yes | `gnosis/sync` |
| `client` | Client to test | Yes | `nethermind-gnosis` |
| `client_config` | Client configuration in YAML format | No | - |
| `extra_flags` | Additional flags to pass to hive | No | - |
| `skip_tests` | Skip tests. Useful when used together with input.website_upload = "true" to upload the website without running tests. | No | `false` |

### Hive Configuration

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `hive_repository` | Hive repository to use | No | `gnosischain/hive` |
| `hive_version` | Hive version/branch to use | No | `master` |
| `go_version` | Go version used to build hive | No | `1.24` |
| `docker_version` | Docker version to use | No | `latest` |

### GCS Upload Configuration

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `gcs_upload` | Upload test results to GCS | No | `false` |
| `gcs_bucket` | GCS bucket name | No* | - |
| `gcs_path` | Path prefix in GCS bucket | No | `''` |
| `gcs_public_url` | Public URL prefix for Hive UI. Used to generate links to detailed results in the summary. | No | `''` |
| `rclone_version` | Rclone version to use | No | `latest` |
| `rclone_config` | Base64 encoded rclone config file | No | - |
| `website_upload` | Upload Hive View website to GCS | No | `true` |
| `website_listing_limit` | The amount of listings to generate for the website index | No | `2000` |
| `website_index_generation` | (Re)generate the test results index for the website | No | `true` |

*Required if `gcs_upload` is `true`

### GitHub Workflow Artifact Configuration

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `workflow_artifact_upload` | Upload test results as a workflow artifact | No | `false` |
| `workflow_artifact_prefix` | Name of the workflow prefix. If not provided, the prefix will be the simulator and client name. | No | - |

## Examples

### Simple example doing sync tests with the latest nethermind-gnosis

```yaml
- uses: gnosischain/hive-github-action@master
  with:
    client: nethermind-gnosis
    simulator: gnosis/sync
```

### With custom client config

You can customize the [client configuration](https://github.com/gnosischain/hive/blob/master/docs/commandline.md#client-build-parameters). This allows using different docker files that are available for each client and their respective build arguments.

```yaml
env:
  CLIENT_CONFIG: |
    - client: nethermind-gnosis
      nametag: prague-devnet-5
      dockerfile: git
      build_args:
        github: NethermindEth/nethermind
        tag: my-custom-branch
```

Then you can use the `CLIENT_CONFIG` environment variable in your workflow.

```yaml
- uses: gnosischain/hive-github-action@master
  with:
    client: nethermind-gnosis
    simulator: gnosis/sync
    client_config: ${{ env.CLIENT_CONFIG }}
```

### Uploading the results directory to GCS

You'll need to create an rclone config for Google Cloud Storage and base64 encode it. Then store it as a GitHub Actions secret on your repository.

An example rclone config for GCS using inline service account credentials:

```toml
# Content of rclone.conf
[gcs]
type = google cloud storage
project_number = my-gcp-project
service_account_credentials = {"type":"service_account","project_id":"...","private_key":"...","client_email":"...","...":"..."}
object_acl = private
bucket_acl = private
location = us-central1
```

Then you can run `base64 -w 0 rclone.conf` (or `base64 -i rclone.conf` on macOS) and store the output as a GitHub Actions secret.

Afterwards you just need to reference the secret for the `rclone_config` input.

```yaml
- uses: gnosischain/hive-github-action@master
  with:
    client: nethermind-gnosis
    simulator: gnosis/sync
    client_config: ${{ env.CLIENT_CONFIG }}
    gcs_upload: true
    gcs_bucket: my-bucket
    gcs_path: my-path
    rclone_config: ${{ secrets.RCLONE_CONFIG }}
```

### Just upload the website using hiveview without running tests

```yaml
- uses: gnosischain/hive-github-action@master
  with:
    website_upload: true # This is required to upload the website
    skip_tests: true # This is required to skip the tests
    gcs_upload: true
    gcs_bucket: my-bucket
    gcs_path: my-path
    rclone_config: ${{ secrets.RCLONE_CONFIG }}
```

### Uploading the results directory as a GitHub workflow artifact

This will upload the test results as a workflow artifact. By default the artifact prefix will be the simulator and client name. You can override this by providing a `workflow_artifact_prefix` input.
```yaml
- uses: gnosischain/hive-github-action@master
  with:
    client: nethermind-gnosis
    simulator: gnosis/sync
    workflow_artifact_upload: true
    # workflow_artifact_prefix: my-custom-prefix
```

## Local testing and development

To test the action locally, you can run [act](https://github.com/nektos/act) with the following command:

```bash
act -s GITHUB_TOKEN="$(gh auth token)" -W '.github/workflows/test.yaml'
```

Or if you configured a custom runner, you can run `act` with the following command:

```bash
act -s GITHUB_TOKEN="$(gh auth token)" -W '.github/workflows/your-workflow.yaml' -P your-self-hosted-runner=catthehacker/ubuntu:act-latest
```

## License

Refer to the repository's license file for information on the licensing of this GitHub Action and the associated software.
