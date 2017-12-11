# Release Notes

!!! note
	Cloud Foundry Container Runtime (CFCR) was formerly known as Kubo, and some CFCR assets still use the Kubo name.

## v0.10.0

**Release Date:** December 8, 2017

* New property `addons_spec_path` in `director.yml`. Operators can use it to provide a K8s spec file that will be applied to the cluster when it comes up -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/152647926).
* New property `worker_count` in `director.yml`. Operators can use it configure the number of K8s workers -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/153055729).
* Enabled the K8s aggregation layer to support API server extensions -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/152931645).
* CFCR was tested to run with 20 workers and 'chatty' workloads -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/153171519).
* Exposed the K8s API connection properties via a BOSH link -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/153321194).
* **[ACTION REQUIRED]** The HAProxy (`proxy`) routing mode is not longer supported -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/152231249).
* **[ACTION REQUIRED]** **GCP:** The `service_account` property in `director.yml` is no longer supported. It has been replaced by `service_account_master` and `service_account_worker` which can be used to reference GCP service accounts that will be provided to master and worker VMs separately -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/150975060).
* **GCP:** New properties `service_key_master` and `service_key_worker` in `director.yml`. Operators can use them to enable the K8s cloud provider use a GCP service account without having to change the BOSH cloud config -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/153171508).
* **GCP:** The GCP K8s Service Catalog was tested on CFCR -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/153293570).
* **GCP:** Predefined a `standard` storage class to be applied when CFCR is deployed on GCP. It uses the `gce-pd` PV provisioner -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/152853461).

### Community contributions 

* GCP bastion has a recent Ubuntu image -- [Deployment #230](https://github.com/cloudfoundry-incubator/kubo-deployment/pull/230).
  Thanks [@alex-slynko](https://github.com/alex-slynko).
* Change `common_name` for the Docker cert -- [Deployment #229](https://github.com/cloudfoundry-incubator/kubo-deployment/pull/229).
  Thanks [@alex-slynko](https://github.com/alex-slynko).
* Support the `nats` link that is already implemented in template -- [Release #134](https://github.com/cloudfoundry-incubator/kubo-release/pull/134).
  Thanks [@drnic](https://github.com/drnic).
* Add namespaces to the cluster, creds and context in `set_kubeconfig` -- [Deployment #235](https://github.com/cloudfoundry-incubator/kubo-deployment/pull/235).
* InfluxDB is not exposed via a `NodePort` anymore -- [Release #138](https://github.com/cloudfoundry-incubator/kubo-release/issues/138).
* Fix in `route-sync` to avoid memory leak -- [Release #140](https://github.com/cloudfoundry-incubator/kubo-release/issues/140).


### Component Versions

The following table lists the component versions for CFCR v0.10.0:

 <table>
  <thead>
  <tr>
    <th>Component</th>
    <th>Version</th>
  </tr>
  </thead>
  <tbody>
  <tr>
    <td>Kubernetes</td>
    <td>1.8.4</td>
  </tr>
  <tr>
    <td>Flannel</td>
    <td>0.5.5</td>
  </tr>
   <tr>
    <td>ETCD</td>
     <td>3.2.10</td>
  </tr>   
  <tr>
    <td>Docker</td>
    <td>1.13.1</td>
  </tr>
  <tr>
    <td>CNI</td>
    <td>0.5.2</td>
  </tr>
  <tr>
    <td>Stemcell</td>
    <td>3445.11</td>
  </tr>
  </tbody>
  </table>
  
## v0.9.0

**Release Date**: November 22, 2017

### Features
* CFCR has been added to the [Certified Kubernetes Conformance Program](https://www.cncf.io/certification/software-conformance/). See the [Conformance Test Results](#conformance-tests-results) below.
* The Docker BOSH release has been updated to v30.1.4.
* The ETCD and master nodes are colocated on the same VM. Deployments of v0.9.0+ have 3 worker nodes and 1 master/ETCD node.
* BOSH has been updated to v264.1.
* The Kubernetes Dashboard is accessible with RBAC mode as cluster admin. The Dashboard needs to be exposed via `kubectl proxy`.

### Bug Fixes
* Dashboard crashing after deployment [GitHub issue #227](https://github.com/cloudfoundry-incubator/kubo-deployment/issues/227)

### Component Versions
The following table lists the component versions for CFCR v0.9.0:

 <table>
  <thead>
  <tr>
    <th>Component</th>
    <th>Version</th>
  </tr>
  </thead>
  <tbody>
  <tr>
    <td>Kubernetes</td>
    <td>1.8.2</td>
  </tr>
  <tr>
    <td>Flannel</td>
    <td>0.5.5</td>
  </tr>
   <tr>
    <td>ETCD</td>
     <td>3.1.8</td>
  </tr>   
  <tr>
    <td>Docker</td>
    <td>1.13.1</td>
  </tr>
  <tr>
    <td>CNI</td>
    <td>0.5.2</td>
  </tr>
  <tr>
    <td>Stemcell</td>
    <td>3445.11</td>
  </tr>
  </tbody>
  </table>

### Conformance Tests Results

[Download](https://storage.googleapis.com/conformance-results/conformance-results-0.9.0-dev.40.tar.gz) the conformance test results.

## v0.8.1

**Release Date**: November 10, 2017

### Features
* Upgraded Kubernetes version to v1.8.2

### Bug Fixes
* Bug in authorization switch mechanism: all clusters were deployed in RBAC by default. New property in `director.yml` to set desired mode (ABAC|RBAC)

### Component Versions
The following table lists the component versions for CFCR v0.8.1:

  <table>
  <thead>
  <tr>
    <th>Component</th>
    <th>Version</th>
  </tr>
  </thead>
  <tbody>
  <tr>
    <td>Kubernetes</td>
    <td>1.8.2</td>
  </tr>
  <tr>
    <td>Flannel</td>
    <td>0.5.5</td>
  </tr>
   <tr>
    <td>ETCD</td>
     <td>3.1.8</td>
  </tr>   
  <tr>
    <td>Docker</td>
    <td>1.11.0</td>
  </tr>
  <tr>
    <td>CNI</td>
    <td>0.5.2</td>
  </tr>
  <tr>
    <td>Stemcell</td>
    <td>3445.11</td>
  </tr>
  </tbody>
  </table>


## v0.8.0

**Release Date**: November 3, 2017

### Features
* Upgraded Kubernetes version to v1.8.1
* Bosh DNS replaces Power DNS
* Memory limit is configurable
* Kubelet resource reservation flags exposed: `kube-reserved`, `system-reserved`, `eviction-hard`. See [Kubernetes docs](https://kubernetes.io/docs/tasks/administer-cluster/reserve-compute-resources/) for more info
* Internal routing from workers to masters through Bosh DNS (no need for HAProxy or LB to route cluster internal traffic) 
* User can load balance traffic from external load balancers

### Improvements
* Removed `worker_node_tag` property to set worker tags automatically for GCP load balancers

### Bug Fixes
* Support for bosh-lite: [GitHub issue #109](https://github.com/cloudfoundry-incubator/kubo-release/issues/109)
* Config file missing for vSphere: [GitHub issue #110](https://github.com/cloudfoundry-incubator/kubo-release/issues/110)

### Component Versions
The following table lists the component versions for CFCR v0.8.0:

  <table>
  <thead>
  <tr>
    <th>Component</th>
    <th>Version</th>
  </tr>
  </thead>
  <tbody>
  <tr>
    <td>Kubernetes</td>
    <td>1.8.1</td>
  </tr>
  <tr>
    <td>Flannel</td>
    <td>0.5.5</td>
  </tr>
   <tr>
    <td>ETCD</td>
     <td>3.1.8</td>
  </tr>   
  <tr>
    <td>Docker</td>
    <td>1.11.0</td>
  </tr>
  <tr>
    <td>CNI</td>
    <td>0.5.2</td>
  </tr>
  <tr>
    <td>Stemcell</td>
    <td>3445.11</td>
  </tr>
  </tbody>
  </table>

### Upgrading from v0.7.0

Perform the following steps to upgrade an existing CFCR v0.7.0 cluster to v0.8.0:

1. Clone the new version of [kubo-deployment](https://github.com/cloudfoundry-incubator/kubo-deployment).
2. Log in to the CredHub server on your BOSH Director with the [CredHub CLI](https://github.com/cloudfoundry-incubator/credhub-cli).
3. Delete the current Kubernetes certificate from CredHub:
	<p class="terminal">$ credhub delete -n "\${director_name}/\${deployment_name}/tls-kubernetes"</p>
3. Verify that the appropriate stemcell is installed in BOSH. To view the uploaded stemcells, run the following command.
	<p class="terminal">$ bosh stemcells</p>
	To upload a new stemcell, run `bosh upload stemcell STEMCELL_URL`.
4. See [Deploying Bosh Director](https://docs-kubo.cfapps.io/installing/gcp/deploying-bosh-gcp/#step-5-deploy-bosh-director) for information on how to update the BOSH Director.
5. See [Deploying CFCR](https://docs-kubo.cfapps.io/installing/deploying-kubo/#step-3-deploy-cfcr) for information on how to upgrade your CFCR cluster.

## v0.7.0

**Release Date**: September 7, 2017

### Features
* `kubo-release` tarball bundled with `kubo-deployment`
* The `deploy-k8s` script deploys local release by default
* Cluster self-healing capabilities enables the recoveru of worker VMs
* Support for persistent volumes in GCP, AWS, and vSphere
* Improved documentation to install on GCP and AWS

### Bug Fixes
* Removed AWS-related tags for other platforms
* Password issues in vSphere: [GitHub issue #102](https://github.com/cloudfoundry-incubator/kubo-release/issues/102)
