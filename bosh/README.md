# PostgreSQL BOSH release

These are sample manifest to bosh create a standalone PostgreSQL.
You can then use it with the service broker.

## bosh-cli

This is assumed to be used with `bosh-cli interpolate`
Get the cli from
```
wget https://s3.amazonaws.com/bosh-cli-artifacts/cli-current-version
wget https://s3.amazonaws.com/bosh-cli-artifacts/bosh-cli-2.0.12-linux-amd64
```
## Create and deploy the bosh release

Edit local.yml to fit what you need for AZ and service network or bosh deployment name.
You may skip the `upload release` if your bosh deployment already has a postgres-release.

```
bosh upload release https://bosh.io/d/github.com/cloudfoundry/postgres-release
./bosh-cli interpolate template.yml -l local.yml > manifest.yml
bosh deployment manifest.yml
bosh deploy
```

