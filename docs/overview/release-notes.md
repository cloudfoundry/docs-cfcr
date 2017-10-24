#Release Notes

## v0.8

**Release Date**: November xx, 2017

## Features
* Upgraded Kubernetes version to 1.8.1
* Upgraded to CredHub 1.0.0
* Bosh DNS replaces power dns
* Virtual memory areas are configurable
* Kubelet resource reservation flags exposed
* Added support for RBAC authorisation mode in Kubernetes (default mode is still ABAC)
* Internal routing from workers to masters through Bosh DNS (no need for HAProxy or LB to route cluster internal traffic) 

## Bug Fixes
* Removed ```worker_node_tag``` property to set worker tags automatically for GCP load balancers
* Support for bosh-lite [GitHub issue #109](https://github.com/cloudfoundry-incubator/kubo-release/issues/109)
* Config file missing for vSphere [GitHub issue #110](https://github.com/cloudfoundry-incubator/kubo-release/issues/110)

## Component Versions
The following table lists the component versions for Kubo v1.0.

  <table>
  <thead>
  <tr>
    <th>Component</th>
    <th>Version</th>
  </tr>
  </thead>
  <tbody>
  <tr>
    <td>Kubernetes</td><td>Version number</td>
  </tr>
  <tr>
    <td>Flannel</td><td>Version number</td>
  </tr>
  <tr>
    <td>Routesync</td><td>Version number</td>
  </tr>
  <tr>
    <td>Flannel</td><td>Version number</td>
  </tr>
   <tr>
    <td>ETCD</td><td>Version number</td>
  </tr>
  </tbody>
  </table>

## Upgrading from v0.7
The following are the steps to execute to upgrade an existing Kubo v0.7 cluster to the new v0.8:

### Update your BOSH director to get the latest components for Kubo v0.8
1. Clone the new version of [kubo-deployment](https://github.com/cloudfoundry-incubator/kubo-deployment)
2. Log into Credhub
3. Delete the current Kubernetes certificate from CredHub
``` credhub delete -n "${director_name}/${deployment_name}/tls-kubernetes" ```

3. Verify stem cell 3445.11 is installed in bosh
4. Update the bosh director
See [Deploying Bosh Director](https://docs-kubo.cfapps.io/installing/gcp/deploying-bosh-gcp/#step-5-deploy-bosh-director)

### Upgrade your Kubo cluster
See [Deploying Kubo](https://docs-kubo.cfapps.io/installing/deploying-kubo/#step-3-deploy-kubo)

