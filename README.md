# BOSH Release for RabbitMQ Enterprise

This repository contains a BOSH release for RabbitMQ Enterprise.

## Updating the RabbitMQ Enterprise version

This guide describes step-by-step the procedure to upgrade RabbitMQ Enterprise version in the BOSH release.

### Prerequisites

Before creating a new BOSH release with the updated binaries, please consider reviewing the official RabbitMQ documentation. It is recommended to become acquainted with all possible upgrade paths and to take note of any potential breaking changes.
<https://www.rabbitmq.com/docs/upgrade>

Breaking changes are documented in the release notes:
<https://github.com/rabbitmq/rabbitmq-server/releases>.

### Add blobs

Use the bosh-cli to execute these steps in the git repo: <https://bosh.io/docs/cli-v2-install>

RabbitMQ binaries can be downloaded from <https://github.com/rabbitmq/rabbitmq-server/releases>,
while Erlang releases from <https://github.com/erlang/otp/releases>

Download the binaries with the following format:

```bash
# rabbitmq-server
rabbitmq-server-generic-unix-x.y.z.tar.xz

# Erlang
otp_src_x.y.z.zz.tar.gz
```

Add binaries as a blob to the BOSH release and upload them to S3.

Before running `upload-blobs`, configure your S3 credentials as shown in `config/private.yml.example`.

For RabbitMQ server:

```bash
# adapt version as needed
bosh add-blob ~/Downloads/rabbitmq-server-generic-unix-3.13.7.tar.xz rabbitmq-server-3.13/rabbitmq-server-generic-unix-3.13.7.tar.xz
bosh upload-blob
```

For Erlang:

```bash
# adapt version as needed
bosh add-blob ~/Downloads/otp_src_26.2.5.11.tar.gz erlang-26/otp_src_26.2.5.11.tar.gz
bosh upload-blob
```

### Create package files

Copy the current `rabbitmq-server` package directory to a new package directory. The new package directory should look like this:

```bash
packages/
├── rabbitmq-server-<x.y>/
│   ├── packaging
│   ├── spec
```

Update all versions in both the `packaging` and `spec` files. The version in those files should be the same as the version of the blob that was added in the previous step.

### Set new version as default

In the `jobs/rabbitmq-server/spec` file, set the `version` to the new version. This will be the default version of RabbitMQ that will be used when deploying the BOSH release. Also add the new package to the `packages` section.

```yaml
packages:
 - erlang-26
 - rabbitmq-server-3.x
 - rabbitmq-server-3.y
...
properties:
   rabbitmq-server.version:
     description: "Version of RabbitMQ to use"
     default: "3.y"
```

### Decommission old versions

This BOSH release will usually include two versions of RabbitMQ: the current version and the upcoming or previous version. When adding a new version you can decommission in parallel any unused versions.
For this :

1. Remove the old blob with following command:

    ```bash
    bosh remove-blob rabbitmq-server-<x.y>/rabbitmq-server-generic-unix-<x.y.z>.tar.xz
    ```

2. Remove the old package directory:

    ```bash
    rm -rf packages/rabbitmq-server-<x.y>
    ```

3. Remove the old version from the `jobs/rabbitmq-server/spec` file.
