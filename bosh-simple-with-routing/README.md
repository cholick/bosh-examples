## bosh-simple-with-routing

This adds
[routingrelease](https://github.com/cloudfoundry-incubator/routing-release)
to `bosh-simple`. Routing release claims a route in Cloud Foundry. Without this,
the Spacebars VM is assigned a static, internal IP. By broadcasting this route,
apps can access the Spacebears service via DNS, and it's also accessible publicly
through the router.

### Setting up release

(`deploy.sh` automates these steps)

Get the Golang distribution and add to blobs

```bash
export go_pkg_remote=https://storage.googleapis.com/golang/go1.9.linux-amd64.tar.gz

wget "${go_pkg_remote}" -O ./tmp/go-linux-amd64.tar.gz
echo "${go_pkg_remote}" > ./tmp/go-version.txt

bosh add-blob ./tmp/go-linux-amd64.tar.gz go-linux-amd64.tar.gz
bosh add-blob ./tmp/go-version.txt go-version.txt
```

Package up our source code and add to blobs
```bash
tar -cvzf ./tmp/spacebears_src.tgz -C ../src/ spacebears/

bosh add-blob ./tmp/spacebears_src.tgz spacebears_src.tgz
```

Create & upload release
```bash
bosh create-release --force
bosh upload-release
```

Download the [routingrelease](https://github.com/cloudfoundry-incubator/routing-release) and upload to the director
```bash
export routing_release_remote=https://github.com/cloudfoundry-incubator/routing-release/releases/download/0.162.0/routing-0.162.0.tgz
export routing_release_path=./tmp/routing-release.tgz

wget "${routing_release_remote}" -O ${routing_release_path}
```

### Deploy (Lite)

If this is a fresh environment, be sure to upload a proper stemcell
```bash
bosh upload-stemcell https://s3.amazonaws.com/bosh-core-stemcells/warden/bosh-stemcell-3445.7-warden-boshlite-ubuntu-trusty-go_agent.tgz
```

The bosh deployment manifest in `manifest/lite_manifest.yml` is setup to work
with a default [deployed BOSH Lite](https://bosh.io/docs/bosh-lite) using
the [cf-release bosh-lite cloud-config](https://github.com/cloudfoundry/cf-deployment/blob/master/iaas-support/bosh-lite/cloud-config.yml)

Deploy
```bash
bosh -d bosh_simple_with_routing deploy manifests/lite_manifest.yml --no-redact
```

#### Troubleshooting
```bash
bosh -d bosh_simple_with_routing ssh spacebears_db_node
# hope over to root for monit and other commands
sudo su -
```

* job logs
    * `/var/vcap/sys/log/spacebears/spacebears.out.log`
    * `/var/vcap/sys/log/spacebears/spacebears.err.log`
* monit logs
    * `/var/vcap/monit/monit.log`

###  Cleanup

```bash
bosh -d bosh_simple_with_routing delete-deployment --force
bosh delete-release bosh-simple-with-routing-spacebears
bosh clean-up
```
