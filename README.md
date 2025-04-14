# BOSH Release for RabbitMQ Enterprise
This repository contains a BOSH release for RabbitMQ Enterprise. 

## Updating the RabbitMQ Enterprise version
In this guide we describe the step-by-step procedure to upgrade the RabbitMQ Enterprise version in the BOSH release.

### Prerequisites
Before creating a new bosh release with new binaries please consider review official RabbitMQ documentation in order to follow possible upgrade paths and note any potential breaking change.
<https://www.rabbitmq.com/docs/upgrade>

Breaking changes are documented in the release notes:
<https://github.com/rabbitmq/rabbitmq-server/releases>.

### Add blobs

Use the bosh-cli to execute these steps in the git repo. <https://bosh.io/docs/cli-v2-install>
Download RabbitMQ binaries from <https://github.com/rabbitmq/rabbitmq-server/releases>.

Download the rabbitmq-server binaries with the following format:
```
rabbitmq-server-generic-unix-<x.y.z>.tar.xz
```

Add binaries as a blob to the bosh release and upload them to s3:

```bash
bosh add-blob ~/Downloads/rabbitmq-server-generic-unix-3.13.7.tar.xz rabbitmq-server-3.13/rabbitmq-server-generic-unix-3.13.7.tar.xz
bosh upload-blob
```


### Create package files
Copy the current `rabbitmq-server` package directory to the a new package directory. The new package directory should look like this:

```bash
packages/
├── rabbitmq-server-<x.y>/
│   ├── packaging
│   ├── spec
```
Update all versions in both the `packaging` and `spec` files. The version in those files should be the same as the version of the blob you added in the previous step.

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
This bosh release will usually include two versions of RabbitMQ: the current version and the newer version. When adding a new version you can decommission a the same time a version that is no longer used anymore.
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