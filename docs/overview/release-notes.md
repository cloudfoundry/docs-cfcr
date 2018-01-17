# Release Notes

!!! note
	Cloud Foundry Container Runtime (CFCR) was formerly known as **Kubo**. Some CFCR assets still use the Kubo name.
	
## v0.12.0

**Release Date:** January 10, 2017

[Download](https://github.com/cloudfoundry-incubator/kubo-deployment/releases/download/v0.12.0/kubo-deployment-0.12.0.tgz) the release artifact.

* Use Kubernetes 1.8.6 -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/154101118).
* Enable secure access to the Dashboard via a `NodePort` when using RBAC -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/154051302).
* Privileged container support is turned off by default. There is a new property named `allow_privileged_containers` in `director.yml` which can be used to enable the feature -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/153848167).
  - also [cloudfoundry-incubator/kubo-deployment#252](https://github.com/cloudfoundry-incubator/kubo-deployment/pull/252) and [cloudfoundry-incubator/kubo-release#153](https://github.com/cloudfoundry-incubator/kubo-release/pull/153)
* Don't update the master node when scaling up workers -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/154036901).
* Switch to use `cfcr.internal` as a TLD instead of `.kubo` -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/153849646).
* Disable all profiling / tracing endpoints by default -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/154051304).
* Always validate `ServiceAccount` tokens exist in etcd as part of authentication -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/153592391).
* Stop the Kubernetes API Server from serving unsecured and unauthenticated access in localhost -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/153592275). 
* Remove unnecessary flag from `kube-proxy` -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/150079004).
* Make applying addon specs not fail if the specs are empty -- [story](https://www.pivotaltracker.com/story/show/154030804) / [cloudfoundry-incubator/kubo-release#150](https://github.com/cloudfoundry-incubator/kubo-release/issues/150).
* Bump system specs timeout to work with slower environments -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/154051298).
* Implement logic to never lose more than one worker nodes during update -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/154051299).
* Restrict the data directory permissions for etcd -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/153592400).
* Use SSL for etcd peer connections -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/153782872).
  - We are currently running single-node etcd clusters so no peer connections are established. Nevertheless, etcd would listen for peer connections over plain HTTP.
* **Openstack:** `openstack_tenant` is not required in the `director.yml` as it is obsolete for OpenStack Keystone v3. The property still exists as it is needed by OpenStack Keystone v2 -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/153137960).
* **Openstack:** VMs deleted from the IaaS in OpenStack do not appear as ghost nodes -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/153816342).
  - The fix for this issue introduces new `director.yml` properties as the OpenStack K8s Cloud Provider needs to be configured: 
     * `auth_url`, `openstack_username`, `openstack_password`, `openstack_project_id`, `openstack_domain`
   - **Caveat:** the BOSH director needs to have `human_readable_vm_names` set to `false` in order for the Kubelet to register with the API Server successfully. See K8s issue: [kubernetes/kubernetes#57765](https://github.com/kubernetes/kubernetes/issues/57765).
* **Fix:** `deploy_k8s` to fail when the addon specs are not successfully applied -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/153401157).
* **Fix:** regression in `abac` authorization mode -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/153875025).
* **vSphere** **PR** Support CFCR deployments on vSphere environments with multiple datacenters -- [cloudfoundry-incubator/kubo-release#127](https://github.com/cloudfoundry-incubator/kubo-release/issues/127) / [cloudfoundry-incubator/kubo-release#148](https://github.com/cloudfoundry-incubator/kubo-release/pull/148).

## BOSH Release

* Rename BOSH jobs to better reflect the Kubernetes processes names -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/151088854).
* **PR** Expose parameters for logging level change -- [cloudfoundry-incubator/kubo-release#151](https://github.com/cloudfoundry-incubator/kubo-release/pull/151).
* **PR** Expose Block Storage parameters for Openstack -- [cloudfoundry-incubator/kubo-release#152](https://github.com/cloudfoundry-incubator/kubo-release/pull/152).

### Component Versions

The following table lists the component versions for CFCR v0.11.0:

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
    <td>1.8.6</td>
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
    <td>3468.13</td>
  </tr>
  </tbody>
  </table>
  
### Conformance Tests Results

[Download](https://storage.googleapis.com/conformance-results/conformance-results-aws-0.11.1-dev.59.tar.gz) the conformance test results.

## v0.11.0

**Release Date:** December 20, 2017.
[Download](https://github.com/cloudfoundry-incubator/kubo-deployment/releases/download/v0.11.0/kubo-deployment-0.11.0.tgz) the release artifact.

* Rename the CF Routing properties in `director.yml` to follow a consistent naming style. [story](https://www.pivotaltracker.com/n/projects/2093412/stories/153171539).
    - **[ACTION REQUIRED]**: If you use CF Routing in CFCR, you must update your `director.yml` with the new CF Routing property names.
* Implement ability to deploy CFCR on an existing BOSH Director and make the experience for BOSH-native users comparable to the one in cf-deployment, bosh-deployment, and concourse-deployment. [PR](https://github.com/cloudfoundry-incubator/kubo-deployment/pull/237).
    - **[ACTION REQUIRED]**: If you deploy CFCR by means of `bosh deploy`, you should re-examine the names of the manifest and the ops-files as various changes have been made.
* **vSphere**: Support persistent workloads on vSphere environments that do not use Resource Pools. Resource Pools are optional in vSphere and CFCF already supports vSphere environments that make use of Resource Pools.  [story](https://www.pivotaltracker.com/n/projects/2093412/stories/153646184).
* Looked into an issue with the `kubernetes-system-spec` post-start script.  [story](https://www.pivotaltracker.com/n/projects/2093412/stories/153535114) / [Github Issue](https://github.com/cloudfoundry-incubator/kubo-release/issues/143).

### Theme: Security

* **AWS**: Reduce the permissions of the AWS IAM policies for the master and worker nodes so that if credentials leak from an AWS CFCR node, the permissions associated with it are minimal. [story](https://www.pivotaltracker.com/n/projects/2093412/stories/151445255).
    - **[ACTION REQUIRED]**: If you deploy CFCR on AWS, you must re-run Terraform to update the IAM Policies.
* Use stemcell version 3468.13.  [story](https://www.pivotaltracker.com/n/projects/2093412/stories/153557143).
* Disallow anonymous requests to the API server.  [story](https://www.pivotaltracker.com/n/projects/2093412/stories/153561120).
* Alter the permissions of files with sensitive information so that they cannot be read by non-root users. [story](https://www.pivotaltracker.com/n/projects/2093412/stories/152711831).
* Make ETCD only listen to TLS connections so that ETCD-bound traffic in the cluster cannot be sniffed. [story](https://www.pivotaltracker.com/n/projects/2093412/stories/153098701).
* Disallow `exec` and `attach` commands to privileged pods. [story](https://www.pivotaltracker.com/n/projects/2093412/stories/153592296).
* **vSphere**: Escape back-slashes for vSphere users in cloud config. [PR](https://github.com/cloudfoundry-incubator/kubo-release/pull/121).

### Component Versions

The following table lists the component versions for CFCR v0.11.0:

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
    <td>3468.13</td>
  </tr>
  </tbody>
  </table>
  
### Conformance Tests Results

[Download](https://storage.googleapis.com/conformance-results/conformance-results-0.11.0-dev.62.tar.gz) the conformance test results.

## v0.10.0

**Release Date:** December 8, 2017.
[Download](https://github.com/cloudfoundry-incubator/kubo-deployment/releases/download/v0.10.0/kubo-deployment-0.10.0.tgz) the release artifact.

* New property `addons_spec_path` in `director.yml`. Operators can use this property to provide a K8s spec file that is applied to the cluster when it comes up. [story](https://www.pivotaltracker.com/n/projects/2093412/stories/152647926).
* New property `worker_count` in `director.yml`. Operators can use this property to configure the number of K8s workers. [story](https://www.pivotaltracker.com/n/projects/2093412/stories/153055729).
* Enabled the K8s aggregation layer to support API server extensions. [story](https://www.pivotaltracker.com/n/projects/2093412/stories/152931645).
* CFCR was tested to run with 20 workers and "chatty" workloads. [story](https://www.pivotaltracker.com/n/projects/2093412/stories/153171519).
* Exposed the K8s API connection properties via a BOSH link. [story](https://www.pivotaltracker.com/n/projects/2093412/stories/153321194).
* **[ACTION REQUIRED]**: The HAProxy (`proxy`) routing mode is not longer supported. [story](https://www.pivotaltracker.com/n/projects/2093412/stories/152231249).
* **[ACTION REQUIRED]** **GCP**: The `service_account` property in `director.yml` is no longer supported. It has been replaced by `service_account_master` and `service_account_worker` which can be used to reference GCP service accounts that are provided to master and worker VMs separately. [story](https://www.pivotaltracker.com/n/projects/2093412/stories/150975060).
* **GCP**: New properties `service_key_master` and `service_key_worker` in `director.yml`. Operators can use these properties to enable the K8s cloud provider to use a GCP service account without having to change the BOSH cloud config. [story](https://www.pivotaltracker.com/n/projects/2093412/stories/153171508).
* **GCP:** The GCP K8s Service Catalog was tested on CFCR. [story](https://www.pivotaltracker.com/n/projects/2093412/stories/153293570).
* **GCP:** Predefined a `standard` storage class to be applied when CFCR is deployed on GCP. It uses the `gce-pd` PV provisioner. [story](https://www.pivotaltracker.com/n/projects/2093412/stories/152853461).

### Community contributions 

* GCP bastion has a recent Ubuntu image. [Deployment #230](https://github.com/cloudfoundry-incubator/kubo-deployment/pull/230).
  Thanks [@alex-slynko](https://github.com/alex-slynko).
* Change `common_name` for the Docker certificate. [Deployment #229](https://github.com/cloudfoundry-incubator/kubo-deployment/pull/229).
  Thanks [@alex-slynko](https://github.com/alex-slynko).
* Support the `nats` link that is already implemented in template. [Release #134](https://github.com/cloudfoundry-incubator/kubo-release/pull/134).
  Thanks [@drnic](https://github.com/drnic).
* Add namespaces to the cluster, creds, and context in `set_kubeconfig`. [Deployment #235](https://github.com/cloudfoundry-incubator/kubo-deployment/pull/235).
* InfluxDB is not exposed via a `NodePort` anymore. [Release #138](https://github.com/cloudfoundry-incubator/kubo-release/issues/138).
* Fix in `route-sync` to avoid memory leak. [Release #140](https://github.com/cloudfoundry-incubator/kubo-release/issues/140).

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
    <td>3468.5</td>
  </tr>
  </tbody>
  </table>
  
### Conformance Tests Results

[Download](https://storage.googleapis.com/conformance-results/conformance-results-0.10.0-dev.52.tar.gz) the conformance test results.
  
## v0.9.0

**Release Date**: November 22, 2017.
[Download](https://github.com/cloudfoundry-incubator/kubo-deployment/releases/download/v0.9.0/kubo-deployment-0.9.0.tgz) the release artifact.

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

**Release Date**: November 10, 2017.
[Download](https://github.com/cloudfoundry-incubator/kubo-deployment/releases/download/v0.8.1/kubo-deployment-0.8.1.tgz) the release artifact.

### Features
* Upgraded Kubernetes version to v1.8.2

### Bug Fixes
* Bug in authorization switch mechanism: all clusters were deployed in RBAC by default. New property in `director.yml` to set desired mode (ABAC|RBAC).

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

**Release Date**: November 3, 2017.
[Download](https://github.com/cloudfoundry-incubator/kubo-deployment/releases/download/v0.8.0/kubo-deployment-0.8.0.tgz) the release artifact.

### Features
* Upgraded Kubernetes version to v1.8.1
* Bosh DNS replaces Power DNS
* Memory limit is configurable
* Kubelet resource reservation flags exposed: `kube-reserved`, `system-reserved`, `eviction-hard`. See [Kubernetes docs](https://kubernetes.io/docs/tasks/administer-cluster/reserve-compute-resources/) for more information.
* Internal routing from workers to masters through BOSH DNS -- no need for HAProxy or LB to route cluster internal traffic
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

**Release Date**: September 7, 2017.
[Download](https://github.com/cloudfoundry-incubator/kubo-deployment/releases/download/v0.7.0/kubo-deployment-0.7.0.tgz) the release artifact.

### Features
* `kubo-release` tarball bundled with `kubo-deployment`
* The `deploy-k8s` script deploys local release by default
* Cluster self-healing capabilities enables the recoveru of worker VMs
* Support for persistent volumes in GCP, AWS, and vSphere
* Improved documentation to install on GCP and AWS

### Bug Fixes
* Removed AWS-related tags for other platforms
* Password issues in vSphere: [GitHub issue #102](https://github.com/cloudfoundry-incubator/kubo-release/issues/102)
