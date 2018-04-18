# Preparing BOSH-Lite for CFCR

This topic describes how to prepare BOSH-Lite for Cloud Foundry Container Runtime (CFCR).

##Prerequisites

A BOSH-Lite environment requires the following:

+ Git
+ BOSH CLI
+ VirtualBox

##Step 1: Create a BOSH-Lite Environment

Follow the procedures in the [BOSH documentation](https://bosh.io/docs/bosh-lite.html) to create a BOSH-Lite environment.

You must deploy BOSH-Lite with CredHub.

1. Clone the `bosh-deployment` repo with the following command:
  <p class="terminal">$ git clone https://github.com/cloudfoundry/bosh-deployment</p>
1. Navigate to the `bosh-deployment` repo with the following command:
  <p class="terminal">$ cd bosh-deployment/</p>
1. Deploy BOSH-Lite with the following command:
  <p class="terminal">
$ bosh create-env ./bosh.yml \
  --state ./state.json \
  -o ./virtualbox/cpi.yml \
  -o ./virtualbox/outbound-network.yml \
  -o ./bosh-lite.yml \
  -o ./bosh-lite-runc.yml \
  -o ./jumpbox-user.yml \
  -o ./uaa.yml \
  -o ./credhub.yml \
  --vars-store ./creds.yml \
  -v director_name="bosh-lite-director" \
  -v internal_ip=192.168.50.6 \
  -v internal_gw=192.168.50.1 \
  -v internal_cidr=192.168.50.0/24 \
  -v outbound_network_name=NatNetwork</p>
1. Export the `BOSH_CLIENT` variable with the following command:
  <p class="terminal">$ export BOSH_CLIENT=admin</p>
1. Export the `BOSH_CLIENT_SECRET` with the following command:
  <p class="terminal">$ export BOSH_CLIENT_SECRET=`bosh int ./creds.yml --path /admin_password`</p>
1. Set an alias for your BOSH-Lite environment with the following command:
  <p class="terminal">$> bosh alias-env vbox -e 192.168.50.6 --ca-cert <(bosh int ./creds.yml --path /director_ssl/ca)</p>

##Step 2: Prepare the BOSH Director

To prepare the BOSH Director, upload the following artifacts:

* Stemcell: go to [bosh.io](http://bosh.io/) and download the latest [BOSH-Lite stemcell](http://bosh.io/stemcells/bosh-warden-boshlite-ubuntu-trusty-go_agent). Upload that stemcell to the Director using `bosh upload-stemcell`.
* Kubo release: clone [https://github.com/cloudfoundry-incubator/kubo-release](kubo-release) locally, create a new BOSH release from master (`bosh create-release`) and upload it to the director (`bosh upload-release`).
* Cloud config: clone [https://github.com/cloudfoundry-incubator/kubo-deployment](kubo-deployment) locally and update (`bosh update-cloud-config`) the cloud config with `configurations/virtualbox/cloud-config.yml`.

###Example

```
# Upload the stemcell
$> bosh -e vbox upload-stemcell https://bosh.io/d/stemcells/bosh-warden-boshlite-ubuntu-trusty-go_agent
# Upload Kubo release
$> git clone https://github.com/cloudfoundry-incubator/kubo-release
$> cd kubo-release/
$> bosh -e vbox create-release
$> bosh -e vbox upload-release
# Update the cloud config
$> cd ../
$> git clone https://github.com/cloudfoundry-incubator/kubo-deployment
$> cd kubo-deployment/
$> bosh -e vbox update-cloud-config configurations/virtualbox/cloud-config.yml
$> cd ../
```

# Step 3: Deploy CFCR

Now the director is ready to run CFCR. Simply deploy CFCR using the default manifest (`manifests/cfcr.yml`) of the [https://github.com/cloudfoundry-incubator/kubo-deployment](kubo-deployment).

## Example

```
$> cd kubo-deployment/
$> bosh -e vbox -d cfcr deploy ./manifests/cfcr.yml
```

# Step 4: Connect to CFCR


