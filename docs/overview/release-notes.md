# Release Notes

## 0.8.0

**Release Date**: November xx, 2017

### Features
* Upgraded Kubernetes version to 1.8.1
* Bosh DNS replaces Power DNS
* Memory limit is configurable
* Kubelet resource reservation flags exposed, `kube-reserved`, `system-reserved`, `eviction-hard`. See [Kubernetes docs](https://kubernetes.io/docs/tasks/administer-cluster/reserve-compute-resources/) for more info
* Internal routing from workers to masters through Bosh DNS (no need for HAProxy or LB to route cluster internal traffic) 

### Improvements
* Removed ```worker_node_tag``` property to set worker tags automatically for GCP load balancers

### Bug Fixes
* Support for bosh-lite [GitHub issue #109](https://github.com/cloudfoundry-incubator/kubo-release/issues/109)
* Config file missing for vSphere [GitHub issue #110](https://github.com/cloudfoundry-incubator/kubo-release/issues/110)

### Component Versions
The following table lists the component versions for Kubo 0.8.0

  <table>
  <thead>
  <tr>
    <th>Component</th>
    <th>Version</th>
     <th>Details</th>
  </tr>
  </thead>
  <tbody>
  <tr>
    <td>Kubernetes</td>
    <td>1.8.1</td>
    <td>https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.8.md#v181</td>
  </tr>
  <tr>
    <td>Flannel</td>
    <td>TBC</td>
    <td>Details</td>
  </tr>
   <tr>
    <td>ETCD</td>
     <td>3.1.8</td>
     <td>https://github.com/coreos/etcd/releases/tag/v3.1.8</td>
  </tr>   
  <tr>
    <td>Docker</td>
    <td>TBC</td>
    <td>Details</td>
  </tr>
  <tr>
    <td>CNI</td>
    <td>0.5.2</td>
    <td>https://github.com/containernetworking/cni/releases/tag/v0.5.2</td>
  </tr>
  <tr>
    <td>Stemcell (GCP)</td>
    <td>3445.11</td>
    <td>http://bosh.cloudfoundry.org/stemcells/bosh-google-kvm-ubuntu-trusty-go_agent</td>
  </tr>
  <tr>
    <td>Stemcell (AWS)</td>
    <td>TBC</td>
    <td>http://bosh.cloudfoundry.org/stemcells/bosh-aws-xen-ubuntu-trusty-go_agent</td>
  </tr>
  <tr>
    <td>Stemcell (vSphere)</td>
    <td>TBC</td>
    <td>http://bosh.cloudfoundry.org/stemcells/bosh-vsphere-esxi-ubuntu-trusty-go_agent</td>
  </tr>
     <tr>
    <td>Stemcell (OpenStack)</td>
    <td>TBC</td>
    <td>http://bosh.cloudfoundry.org/stemcells/bosh-openstack-kvm-ubuntu-trusty-go_agent</td>
  </tr>
  </tbody>
  </table>

### Upgrading from 0.7.0
The following are the steps to execute to upgrade an existing Kubo 0.7.0 cluster to the new 0.8.0:

#### Update your BOSH director to get the latest components for Kubo 0.8.0
1. Clone the new version of [kubo-deployment](https://github.com/cloudfoundry-incubator/kubo-deployment)
2. Log into Credhub
3. Delete the current Kubernetes certificate from CredHub
``` credhub delete -n "${director_name}/${deployment_name}/tls-kubernetes" ```

3. Verify the appropriate stemcell is installed in bosh
4. Update the bosh director
See [Deploying Bosh Director](https://docs-kubo.cfapps.io/installing/gcp/deploying-bosh-gcp/#step-5-deploy-bosh-director)

#### Upgrade your Kubo cluster
See [Deploying Kubo](https://docs-kubo.cfapps.io/installing/deploying-kubo/#step-3-deploy-kubo)


## 0.7.0

**Release Date**: September 7, 2017

### Features
* kubo-release tarball bundled with the kubo-deployment
* The deploy-k8s script deploys local release by default
* Cluster self healing capabilities: recover worker vms
* Support for persistent volumes in GCP, AWS and vSphere
* Improved documentation to install on GCP and AWS

### Bug Fixes
* Removed AWS-related tags for other platforms
* Password issues in vSphere [GitHub issue #102](https://github.com/cloudfoundry-incubator/kubo-release/issues/102)
