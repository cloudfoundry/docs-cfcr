# Release Notes

!!! note
	Cloud Foundry Container Runtime (CFCR) was formerly known as **Kubo**. Some CFCR assets still use the Kubo name.

## v0.35.0

**Release Date**: October 1, 2019

### Downloads
* [Release](https://github.com/cloudfoundry-incubator/kubo-release/releases/download/v0.35.0/kubo-0.35.0-ubuntu-xenial-456.27-20191001-214246-343749329.tgz)
* [Deployment manifests](https://github.com/cloudfoundry-incubator/kubo-deployment/releases/download/v0.35.0/kubo-deployment-0.35.0.tgz)

### Features and Updates
* Updated to Stemcell v456.27
* Updated to go1.12.9.linux-amd64
* Updated to kubernetes v1.14.5
* Updated to metrics-server v0.3.5
* Updated to coredns 1.4.0
* Added jobs that were previously part of [kubo-release-windows](https://github.com/cloudfoundry-incubator/kubo-release-windows)

### Bug Fixes

* Fixed a bug which prevented creating persistent volumes on vSphere when there were no resource pools set in the cloud config.

## v0.34.0

**Release Date**: June 21, 2019

### Downloads
* [Release](https://github.com/cloudfoundry-incubator/kubo-release/releases/download/v0.34.0/kubo-0.34.0-ubuntu-xenial-315.41-20190621-181712-51217485.tgz)
* [Deployment manifests](https://github.com/cloudfoundry-incubator/kubo-deployment/releases/download/v0.34.0/kubo-deployment-0.34.0.tgz)

### Features and Updates
* Add flannel etcd certs to Windows
* Configure Kubelet force drain settings
* Allow customization of vxlan VNI and port
* Updated to go1.12.6.linux-amd64
* Updated to Stemcell v315.41

## v0.33.0 (skipped due to internal issue)

**Release Date**: N/A

## v0.32.0

**Release Date**: May 3, 2019

### Downloads
* [Release](https://github.com/cloudfoundry-incubator/kubo-release/releases/download/v0.32.0/kubo-release-0.32.0.tgz)
* [Deployment manifests](https://github.com/cloudfoundry-incubator/kubo-deployment/releases/download/v0.32.0/kubo-deployment-0.32.0.tgz)

### Features and Updates

* Updated to Kubernetes 1.14.1
* Updated to go1.12.4.linux-amd64
* Updated to metrics-server version v0.3.2
* Updated to Stemcell v315.11
* Updated to bpm version 1.0.4
* Enabled user authorization on etcd

## v0.31.0

**Release Date**: March 25, 2019

### Downloads
* [Release](https://github.com/cloudfoundry-incubator/kubo-release/releases/download/v0.31.0/kubo-release-0.31.0.tgz)
* [Deployment manifests](https://github.com/cloudfoundry-incubator/kubo-deployment/releases/download/v0.31.0/kubo-deployment-0.31.0.tgz)

### Features and Updates

* Updated to Kubernetes 1.13.5
* Updated to Stemcell v250.23
* Updated golang to v1.12.1
* Added the ability to deploy Windows Workers through the [kubo-release-windows](https://github.com/cloudfoundry-incubator/kubo-release-windows).
  See [ops-files/windows/add-worker.yml](https://github.com/cloudfoundry-incubator/kubo-deployment/blob/master/manifests/ops-files/windows/add-worker.yml) for details. Currently, `kubo-release-windows`
	is _only_ compatible on vSphere in an internet-connected environment.
* Added [an ops-file](https://github.com/cloudfoundry-incubator/kubo-deployment/blob/master/manifests/ops-files/set-fs-inotify-limit.yml) to set `fs.inotify.max_user_watches`.

### Bug Fixes

* Fixed an issue with unbound variables in HTTP and HTTPS proxy envs for in the Docker job ([commit](https://github.com/cloudfoundry-incubator/docker-boshrelease/commit/346a68729e20598c277d55283034553cfca32acb))
* Set the Kubelet's `root-dir` to a partition that is intended for ephemeral data ([Issue #351](https://github.com/cloudfoundry-incubator/kubo-deployment/issues/351))
* Removed Kubelet's [`labels` property](https://github.com/cloudfoundry-incubator/kubo-release/commit/cea69999be1e96bd2fc0f0fc9da118a085d67a4c)
  and merge [`k8s-args.root-dir` property with default labels](https://github.com/cloudfoundry-incubator/kubo-release/commit/b2e1b0487ad29d6218f63b283d16198655ea04a3)

## v0.30.0

[Download Release](https://github.com/cloudfoundry-incubator/kubo-release/releases/download/v0.30.0/kubo-release-0.30.0.tgz)

[Download Deployment Manifests](https://github.com/cloudfoundry-incubator/kubo-deployment/releases/download/v0.30.0/kubo-deployment-0.30.0.tgz)

**Release Date** Mar 4, 2019

* Bumped Stemcell to v250.9
* Bumped Docker to 18.06.3-ce


**Fix** Quote and escape strings in cloud-provider's ini formatting to accept special characters in cloud provider configuration.

**Fix** Deployment CFCR manifests reference a release of docker with the runc CVE-2019-5736 addressed.

## v0.29.0
[Download](https://github.com/cloudfoundry-incubator/kubo-deployment/releases/download/v0.29.0/kubo-deployment-0.29.0.tgz) the release artifact.

**Release Date:** Feb 14, 2019

* **Important:** Replaced the `etcd` certificates and authorities with a unique CA (`etcd_ca`) to address [CVE-2019-3779](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-3779). **See the [Action Required](#action-required) section for details.**
* Bumped Docker to `v18.06.2-ce`.
* Upgraded `flannel` to `v0.11.0`.
* Upgraded `bpm` to `v1.0.3`.
* Removed `kube-dns` as an option from `apply-addons`.

### Action Required

Users upgrading from earlier versions of CFCR must perform a series of upgrade steps to facilitate a smooth rotation of the `etcd` certificates.

With BOSH, certificate rotations require a three-phase process that generates a CA, generates and utilizes a set of end-user certificates, and retires the original CA.

The three phases are as follows:

1. Generate the new `etcd_ca` and add it to the list of trust anchors for `etcd`.
2. Generate the new client and server certificates from `etcd_ca` and begin using them.
3. Remove the old CA (`kubo_ca`) from `etcd`'s trust anchors.

Perform the steps below.

#### Steps

1. Redeploy the existing CFCR deployment, applying an additional ops-file for phase one:
  ```yaml
  - type: replace
    path: /variables/-
    value:
      name: etcd_ca
      options:
        common_name: etcd.ca
        is_ca: true
      type: certificate

  - type: replace
    path: /instance_groups/name=master/jobs/name=etcd/properties/tls/etcd/ca
    value: ((tls-etcd-v0-17-0.ca))((etcd_ca.certificate))

  - type: replace
    path: /instance_groups/name=master/jobs/name=etcd/properties/tls/etcdctl/ca
    value: ((tls-etcdctl.ca))((etcd_ca.certificate))

  - type: replace
    path: /instance_groups/name=master/jobs/name=etcd/properties/tls/peer/ca
    value: ((tls-etcd-v0-17-0.ca))((etcd_ca.certificate))
  ```

2. Redeploy the existing CFCR deployment, removing the ops-file for phase one and applying the ops-file for phase two:
  ```yaml
  - type: replace
    path: /variables/-
    value:
      name: etcd_ca
      options:
        common_name: etcd.ca
        is_ca: true
      type: certificate

  - type: replace
    path: /variables/-
    value:
      name: tls-etcd-v0-29-0
      options:
        ca: etcd_ca
        common_name: '*.etcd.cfcr.internal'
        extended_key_usage:
          - client_auth
          - server_auth
      type: certificate

  - type: replace
    path: /variables/-
    value:
      name: tls-etcdctl-v0-29-0
      options:
        ca: etcd_ca
        common_name: 'etcdClient'
        extended_key_usage:
          - client_auth
      type: certificate

  - type: replace
    path: /instance_groups/name=master/jobs/name=etcd/properties/tls/etcd
    value:
      ca: ((tls-etcd-v0-17-0.ca))((etcd_ca.certificate))
      certificate: ((tls-etcd-v0-29-0.certificate))
      private_key: ((tls-etcd-v0-29-0.private_key))

  - type: replace
    path: /instance_groups/name=master/jobs/name=etcd/properties/tls/etcdctl
    value:
      ca: ((tls-etcdctl.ca))((etcd_ca.certificate))
      certificate: ((tls-etcdctl-v0-29-0.certificate))
      private_key: ((tls-etcdctl-v0-29-0.private_key))

  - type: replace
    path: /instance_groups/name=master/jobs/name=etcd/properties/tls/peer
    value:
      ca: ((tls-etcd-v0-17-0.ca))((etcd_ca.certificate))
      certificate: ((tls-etcd-v0-29-0.certificate))
      private_key: ((tls-etcd-v0-29-0.private_key))
  ```

3. Upgrade to `v0.29.0`, completing phase three.

## v0.28.0

[Download](https://github.com/cloudfoundry-incubator/kubo-deployment/releases/download/v0.28.0/kubo-deployment-0.28.0.tgz) the release artifact.

**Release Date** Feb 7, 2019

We upgraded to **Kubernetes 1.13.3**

Default stemcell bumped to Xenial v250.4

Added support for AWS tagging using `kubernetes.io/cluster/<cluster_tag>` format

Removed bundling `heapster` and `influxdb`

Bumped version of CoreDNS to 1.3.1

Updated the [system specs](https://github.com/cloudfoundry-incubator/kubo-release/commit/ec188538b569d66ab64f8c2a2ce84d8f20eac414) to be inline with the samples provided for k8s 1.13.3

**DOC FIX**: Changed documentation on [encryption configuration](https://github.com/cloudfoundry-incubator/kubo-deployment/pull/373)

**DOC FIX**: Recommended values for `NO_PROXY` when using [proxy configuration](https://github.com/cloudfoundry-incubator/kubo-release/blob/develop/docs/using-proxy.md)

## v0.27.0
[Download](https://github.com/cloudfoundry-incubator/kubo-deployment/releases/download/v0.27.0/kubo-deployment-0.27.0.tgz) the release artifact.

**Release Date:** Jan 18, 2019

CFCR now defaults to using [CoreDNS](https://coredns.io/) instead of kube-dns - When upgrading you may want to [remove kube-dns](https://github.com/cloudfoundry-incubator/kubo-release/tree/develop#dns)

CFCR now provides an ops-file for [deploying with kube-dns](https://github.com/cloudfoundry-incubator/kubo-deployment/blob/v0.27.0/manifests/ops-files/use-kube-dns.yml)

CFCR now ships with [etcd 3.3.11](https://github.com/cloudfoundry-incubator/cfcr-etcd-release/releases/tag/v1.9.0)

CFCR now ships with [docker-bosh-release 33.0.1](https://github.com/cloudfoundry-incubator/docker-boshrelease/releases/tag/v33.0.1)

CFCR now provides an ops-file for [deploying workers with persistent disks](https://github.com/cloudfoundry-incubator/kubo-deployment/blob/v0.27.0/manifests/ops-files/use-persistent-disk-for-workers.yml)

CFCR now provides documentation for disabling Linux swap.

Default stemcell bumped to Xenial v170.21

**BUG FIX**: Configure [audit logging to preempt log rotation](https://www.pivotaltracker.com/n/projects/2093412/stories/162821963) to avoid corruption of audit logs due to  [kubernetes/kubernetes#52865](https://github.com/kubernetes/kubernetes/issues/52865)

**BUG FIX**: [Unmount overlay2 mounts](https://github.com/cloudfoundry-incubator/docker-boshrelease/issues/125) during docker shutdown

## v0.26.0
[Download](https://github.com/cloudfoundry-incubator/kubo-deployment/releases/download/v0.26.0/kubo-deployment-0.26.0.tgz) the release artifact.

**Release Date:** Dec 19, 2018

We upgraded to **Kubernetes 1.12.4**

We upgraded the Dashboard to v1.10.1, which includes the patch for (CVE-2018-18264)

CFCR now ships with go 1.11.4

CFCR now ships with JQ package 1.6

CFCR now allows [audit policy to be configurable](https://www.pivotaltracker.com/n/projects/2093412/stories/162351243) via the manifest

CFCR now allows [configurable timeout for kubectl drain](https://www.pivotaltracker.com/n/projects/2093412/stories/161739221)

CFCR now ensures that [BOSH DNS will be chosen first](https://www.pivotaltracker.com/n/projects/2093412/stories/162158342) by Kube DNS during resolution

CFCR now ships as a pre-compiled release

CFCR now allows the [audit log flag values to be configured by ops file](https://www.pivotaltracker.com/story/show/162165401)

**vSphere** Wait for all [disks to be detached](https://www.pivotaltracker.com/n/projects/2093412/stories/162494119) before shutting down worker nodes

**GCP** Change [GCP to use hostname override](https://www.pivotaltracker.com/n/projects/2093412/stories/162638723) due to change of behaviour in stemcell 170.13

**BUG FIX**: The [default flag values for audit log](https://www.pivotaltracker.com/story/show/162165401) now allow the log file be handled correctly by BOSH logrotate

## v0.25.0
[Download](https://github.com/cloudfoundry-incubator/kubo-deployment/releases/download/v0.25.0/kubo-deployment-0.25.0.tgz) the release artifact.

**Release Date:** Dec 5, 2018

We upgraded to **Kubernetes 1.12.3**, which includes the patch for (CVE 2018-1002105)

**BREAKING**: Bosh-dns was removed as an addon.  CFCR now assumes it's already in the director's runtime config. The ops-file for using runtime config is now empty and will be removed in future versions of CFCR.

`--allow-privileged` and `--keep-terminated-pod-volumes` flags in kubelet configuration within the default manifest have both been removed since they have both been deprecated.

Added an ops-file to enable data [encryption at rest](https://www.pivotaltracker.com/story/show/162067187).

CFCR now uses pre-compiled releases for kubo-release to make deploys faster.

Added an ops-file the configure the number of [workers](https://www.pivotaltracker.com/story/show/161756564).

Added an ops-file to change [certificate duration](https://www.pivotaltracker.com/story/show/160847352).

CFCR now ships with Docker 18.06.

CFCR versions are now tied to a [specific version](https://www.pivotaltracker.com/story/show/162013927) of a stemcell in manifest to ensure stemcell compatibility.

**Azure**: Added support for [CIFS volumes](https://www.pivotaltracker.com/story/show/161270158) for Azure Files or FlexVolumes.

**Azure**: Azure cloud provider can now use [useManagedIdentityExtension](https://www.pivotaltracker.com/story/show/161338755) property

**BUG FIX**: Azure load balancers now work with default configuration.

**BUG FIX**: Now possible to use services with Local externalTrafficPolicy in vSphere & AWS.

**BUG FIX**: Fixed OIDC ops-file to work with flexible flag.

**BUG FIX**: Fixed missing kubelet logs.

## v0.24.0
[Download](https://github.com/cloudfoundry-incubator/kubo-deployment/releases/download/v0.24.0/kubo-deployment-0.24.0.tgz) the release artifact.

**Release Date:** Nov 9, 2018

**BREAKING CHANGE** In order to expose all of the kubernetes configuration the manifest format has changed. All the properties for Kubernetes jobs, for example _[kube-apiserver](https://www.pivotaltracker.com/story/show/160558973), [kubelet](https://www.pivotaltracker.com/story/show/160885959), [kube-proxy](https://www.pivotaltracker.com/story/show/160896710), [cloud-provider](https://www.pivotaltracker.com/story/show/161421611) etc_. have been placed under `k8s-args` section. Now, every flag that Kubernetes job has can be passed to the Bosh release. All properties in `kubo-deployment` have been changed. If you have custom operation files to modify the manifest, you will need to change them before you upgrade.

In addition to kubernetes job properties being exposed, [cloud provider](https://www.pivotaltracker.com/story/show/161421611) flags are now exposed to allows more flexible configuration. Any configuration in the source cloud provider will be exposed in the release, for all IaaS.

For more information on modifying kubernetes arguments refer to [this page](https://github.com/cloudfoundry-incubator/kubo-release/blob/master/docs/configuring-kubernetes-properties.md) in our docs.

**NOTICE** While all the properties have been exposed in the release, CFCR team does not support properties that are not part of the `kubo-deployment` manifest or ops-files.

### Kube-apiserver Admission Controllers
In exposing the kubernetes configuration, we now expose the **admission-controllers**. We support the kubernetes defaults, and adding the following additional admission controllers: DenyEscalatingExec, SecurityContextDeny and [PodSecurity Policy](https://www.pivotaltracker.com/story/show/155793102).

*  **SECURITY** The DenyEscalatingExec and SecurityContextDeny admission control plugins are no longer enabled by default. This change was made to better align CFCR with the Kubernetes admission controller defaults and to make it easier to for ops-files to incrementally enable or disable the additional plugins. We recommend that you harden your cluster by applying these ops-files to your manifest.:
  - `kubo-deployment/manifests/ops-files/enable-denyescalatingexec.yml`
  - `kubo-deployment/manifests/ops-files/enable-securitycontextdeny.yml `

* **SECURITY** we have modified our system add-ons PodSecurityPolicy controller works with system addons out-of-the-box. To enable the controller and use policies apply the following ops-file `kubo-deployment/manifests/ops-files/enable-podsecuritypolicy.yml`. CFCR **does not** create default Policy for applications. Please, follow the documentation to enable them https://kubernetes.io/docs/concepts/policy/pod-security-policy/. You need to create PodSecurityPolicy in advance if you upgrading the cluster in order to prevent application downtime. This can be done manually or using new `post-start-custom-specs` property on `kubernetes-roles` job.

### Other changes
* It is now possible to **backup and restore** the cluster control plane when there is 3 masters. This was done by updating the BBR scripts in our etcd release. Thank you Platform Recovery Team for their [PR](https://www.pivotaltracker.com/story/show/160512406)

* **Azure** deployment now in Beta stage, with a number of changes in v0.24. Use stemcell from xenial 170 line to deploy it. We expect a number of changes to come in future releases to complete our support coverage.

* We have [exposed the live-restore flag](https://www.pivotaltracker.com/story/show/161166008) in the docker release, and setting this to true in our manifest. This is to improve workload stability in failure scenarios where the docker daemon is restarted.

* In order for us to support NFS Storage Class in Kubernetes, we now install the the nfs-common library. This was removed in the Xenial stemcell line. – [bug](https://www.pivotaltracker.com/story/show/161318475)

### Component Versions

The following table lists the component versions for CFCR v0.24.0:

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
    <td>1.11.3</td>
  </tr>
  <tr>
    <td>Flannel</td>
    <td>0.10.0</td>
  </tr>
   <tr>
    <td>ETCD</td>
     <td>3.3.9</td>
  </tr>
  <tr>
    <td>Docker</td>
    <td>17.12.1-ce</td>
  </tr>
  <tr>
    <td>CNI</td>
    <td>0.7.1</td>
  </tr>
  </tbody>
</table>

## v0.23.0
[Download](https://github.com/cloudfoundry-incubator/kubo-deployment/releases/download/v0.23.0/kubo-deployment-0.23.0.tgz) the release artifact.

**Release Date:** Oct 16, 2018

* We have extended our multi-cloud support to **Azure**.  [PR #223](https://github.com/cloudfoundry-incubator/kubo-release/pull/223)
  _We consider this experimental as we learn, iterate and extended our test infrastructure._

* As previously announced, the route-sync functionality is now removed. [#158559777](https://www.pivotaltracker.com/story/show/158559777)

* In order to help automate the maintenance load balancers for master nodes during deploy and upgrade on GCP, we have added the _backend-service_ in our default vm_extensions - [#160417211](https://www.pivotaltracker.com/story/show/160417211)

* Fix: Fixed an issue seen during drain: [#251](https://github.com/cloudfoundry-incubator/kubo-release/issues/251)

### Component Versions

The following table lists the component versions for CFCR v0.23.0:

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
    <td>1.11.3</td>
  </tr>
  <tr>
    <td>Flannel</td>
    <td>0.10.0</td>
  </tr>
   <tr>
    <td>ETCD</td>
     <td>3.3.9</td>
  </tr>
  <tr>
    <td>Docker</td>
    <td>17.12.1-ce</td>
  </tr>
  <tr>
    <td>CNI</td>
    <td>0.7.1</td>
  </tr>
  </tbody>
</table>

## v0.22.0
[Download](https://github.com/cloudfoundry-incubator/kubo-deployment/releases/download/v0.22.0/kubo-deployment-0.22.0.tgz) the release artifact.

**Release Date:** Sept 28, 2018

* Upgraded to Kubernetes v1.11.3

* In order to allow workloads that have a security context configuration, the SecurityContextDeny admission controller can now be disabled through a ops-file, it will continued to be enabled by default. The allow-privilege property and corresponding ops-file will no longer disable the SecurityContextDeny admission controller.
  _This is a change in behaviour, if you were using allow-privileged to disable SecurityContextDeny, you should now use the disable-security-context-deny.yml ops-file_

* The kubo-release version set in manifest is configured to the most recent released version. It will no longer be _latest_ or bundled within the _kubo-deployment_ release artifact. The _kubo-deployment_ folder name specifies the version.

* For GCP deployment we have allowed for the setting of the sub-network name. Doing this will allow for services using internal loadbalancers to be deployed – [GH issue](https://github.com/cloudfoundry-incubator/kubo-release/issues/183)

* CFCR no longer supplies scripts to deploy BOSH. We assume a BOSH director has been provisioned, with some standard cloud config dependencies. If you need to deploy BOSH we recommend using [BOSH Boot Loader](https://github.com/cloudfoundry/bosh-bootloader). See the guide below for migration away from _deploy_bosh_.

* Added an [ops-file](https://github.com/cloudfoundry-incubator/kubo-deployment/blob/master/manifests/cloud-config/iaas/vsphere/use-vm-extensions.yml) to use vm_extensions to on vSphere, namely disk.enableUUID

* **Documentation** We have provided instructions on how to configure Kubernetes with a Cloud Provider for each Iaas in our [docs](https://github.com/cloudfoundry-incubator/kubo-release/blob/master/docs/cloud-provider.md). The cloud provider interfaces with the IAAS to provision TCP Load Balancers, Nodes, Networking Routes, and Persistent Volumes.

### deploy_bosh Migration Guide

Going forward, we're encouraging users to discontinue using the deprecated deploy_bosh and deploy_k8s scripts. To help with the transition from the scripts to
standard practices for managing bosh deployments, we've outlined a few changes to be aware of:

**Cloud Config**

Previously the _deploy_bosh_ scripts would generate a compatible cloud config for CFCR clusters. The generated cloud-configs are still valid and can be used. See our [documentation](https://github.com/cloudfoundry-incubator/kubo-release#prerequisites) for cloud-config for requirements.
If you are configuring CFCR with a cloud provider, we've moved to using vm_extensions defined with BOSH generic configs. See our cloud-provider [documentation](https://github.com/cloudfoundry-incubator/kubo-release/blob/master/docs/cloud-provider.md) to see how to set this up.

**Deployment Manifest**

Instead of generating a manifest via the _deploy_k8s_ script, we encourage users to follow our deployment docs and create a bosh deploy command with ops-files appropriate for their CFCR clusters.
One of the big reasons we deprecated these scripts was to decouple the bosh director provisioning steps when creating CFCR clusters. Previously, _deploy_bosh_ would create an artifact called _director.yml_ and this file was used in _deploy_k8s_ to provision CFCR clusters. The _director.yml_ file contains credentials that can still be used as vars to bosh deploy command. However the file itself is not required.

### Component Versions

The following table lists the component versions for CFCR v0.22.0:

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
    <td>1.11.3</td>
  </tr>
  <tr>
    <td>Flannel</td>
    <td>0.10.0</td>
  </tr>
   <tr>
    <td>ETCD</td>
     <td>3.3.9</td>
  </tr>
  <tr>
    <td>Docker</td>
    <td>17.12.1-ce</td>
  </tr>
  <tr>
    <td>CNI</td>
    <td>0.7.1</td>
  </tr>
  </tbody>
</table>


## v0.21.0
[Download](https://github.com/cloudfoundry-incubator/kubo-deployment/releases/download/v0.21.0/kubo-deployment-0.21.0.tgz) the release artifact.

**Release Date:** August 29, 2018

!!! note
	The Kubernetes 1.11 release kicked off the deprecation timeline for the Heapster component, see https://github.com/kubernetes/heapster/blob/master/docs/deprecation.md for more info. As a result, we're in the process of replacing Heapster with Metrics Server in an upcoming releases of kubo-release. Once that process is complete, Heapster will no longer be a default addon applied in 'apply-specs' errand.

* We are deploying metrics server by default as part of the _apply-specs_ errand. Heapster is still deployed but will be removed in a future release – [story](https://www.pivotaltracker.com/story/show/157915809)

* With metrics server deployed securely, CFCR supports [Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/). This includes optional HPA configuration flags as part of CFCR release spec. – [story](https://www.pivotaltracker.com/story/show/157915809)

* **Fix:** We found and responded to an issue in kubernetes, which was causing issues with updating StatefulSet workloads on AWS. We have improved our drain behvaviour, and there is a manual workaround – [bug](https://www.pivotaltracker.com/story/show/159732987)
_If you find the rare case, where a pod that uses volumes is stuck in ContainerCreating state after upgrade, also you can see FailedMount message when you run kubectl describe pod <pod name> command. To manually fix it, recreate a worker using bosh recreate worker/<id>_

* We have included CoreDNS as an optional addon. This can be set by applying the _use-coredns.yml_ ops-file before running the apply-specs errand – [story](https://www.pivotaltracker.com/story/show/159696570)
_CoreDNS should be deployed in place of kube-dns. kube-dns deployment should be deleted from existing clusters, after deploying CoreDNS_

* **Fix:** There is an issue deploying v0.20 in environments not connected an internet registry. We fixed the name in a packaged docker image – [bug](https://www.pivotaltracker.com/story/show/159706265)

* We have enabled the HostPort feature, to allow pods to open external ports on the worker node. This includes an upgrade to the CNI component to v0.7.1 – [story](https://www.pivotaltracker.com/story/show/159167067)

* We have incorporated Kubernetes release v1.11.2


### Component Versions

The following table lists the component versions for CFCR v0.21.0:

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
    <td>1.11.2</td>
  </tr>
  <tr>
    <td>Flannel</td>
    <td>0.10.0</td>
  </tr>
   <tr>
    <td>ETCD</td>
     <td>3.3.9</td>
  </tr>
  <tr>
    <td>Docker</td>
    <td>17.12.1-ce</td>
  </tr>
  <tr>
    <td>CNI</td>
    <td>0.7.1</td>
  </tr>
  </tbody>
</table>



## v0.20.0
[Download](https://github.com/cloudfoundry-incubator/kubo-deployment/releases/download/v0.20.0/kubo-deployment-0.20.0.tgz) the release artifact.

**Release Date:** August 8, 2018

**Known Issue** There is an [issue](https://github.com/cloudfoundry-incubator/kubo-release/issues/243) with deploying cfcr v0.20 in airgapped environments, related to an image dependency. This will be fixed in the next release.

### Features

* Upgraded Kubernetes version to v1.11. For more information about the upgrade, see story [#159328985](https://www.pivotaltracker.com/story/show/159328985) in the CFCR Tracker. For more information about Kubernetes v1.11, see the [Kubernetes release notes](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.11.md#kubernetes-111-release-notes).
* Upgraded the default stemcell to Ubuntu 16.04 Xenial. For more information about the upgrade, see story [#158004796](https://www.pivotaltracker.com/story/show/158004796) in the CFCR Tracker.

    !!! tip
        If you used CFCR’s `./deploy_bosh` script to deploy your BOSH director, we recommend updating the BOSH runtime config, as it has a reference to Ubuntu Trusty. For more information, see the [BOSH documentation](https://bosh.io/docs/runtime-config/).

* This release includes BOSH Backup and Restore (BBR) scripts to restore the state of a cluster from a backup. For more information about the feature, see stories [#158945642](https://www.pivotaltracker.com/story/show/158945642) and [#158946553](https://www.pivotaltracker.com/story/show/158946553) in the CFCR Tracker. For more information about BBR, see the [BBR documentation](https://docs.cloudfoundry.org/bbr/).

    !!! note
        The BBR functionality is verified against a single-master cluster running stateless workloads. We will support further verified scenarios in future versions.

* Upgraded the ETCD release to v3.3.9.

* **Deprecation notice**: Heapster is officially deprecated in Kubernetes v1.11. We will remove it by default in favor of `metrics-server` in a future release.

### Component Versions

The following table lists the component versions for CFCR v0.20.0:

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
    <td>1.11.1</td>
  </tr>
  <tr>
    <td>Flannel</td>
    <td>0.10.0</td>
  </tr>
   <tr>
    <td>ETCD</td>
     <td>3.3.9</td>
  </tr>
  <tr>
    <td>Docker</td>
    <td>17.12.1-ce</td>
  </tr>
  <tr>
    <td>CNI</td>
    <td>0.5.2</td>
  </tr>
  </tbody>
</table>


## v0.19.0
[Download](https://github.com/cloudfoundry-incubator/kubo-deployment/releases/download/v0.19.0/kubo-deployment-0.19.0.tgz) the release artifact.

**Release Date:** July 18, 2018

* **Deprecation notice** Future release will remove support of the CF routing feature – [#157695924](https://www.pivotaltracker.com/story/show/157695924)

* Allow CIDRs to be configured for pods and services – [pr #220](https://github.com/cloudfoundry-incubator/kubo-release/pull/220),[#157480131](https://www.pivotaltracker.com/story/show/157480131)
* Update the admission-controllers based on kubernetes recommendations – [#156525910](https://www.pivotaltracker.com/story/show/156525910)
_Added DefaultTolerationSeconds and ValidatingAdmissionWebhook. We removed NamespaceExists, as it is redunant._
* Kubernetes v1.10.5 version – [#158527191](https://www.pivotaltracker.com/story/show/158527191)
* Changed the docker storage driver from overlay to overlay2 – [#158495554](https://www.pivotaltracker.com/story/show/158495554)
_When upgrading, the old images will remain on each worker in the /var/vcap/data/docker/docker/overlay directory.
Recommended mitigation is to manually delete the directory after upgrading_
* Allow NTLM formatted usernames for vSphere – [pr #229](https://github.com/cloudfoundry-incubator/kubo-release/pull/229)
* **Fix:** improve drain script for upgrades on large clusters – [#158782574](https://www.pivotaltracker.com/story/show/158782574)

### Component Versions

The following table lists the component versions for CFCR v0.19.0:

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
    <td>1.10.5</td>
  </tr>
  <tr>
    <td>Flannel</td>
    <td>0.10.0</td>
  </tr>
   <tr>
    <td>ETCD</td>
     <td>3.3.1</td>
  </tr>
  <tr>
    <td>Docker</td>
    <td>17.12.1-ce</td>
  </tr>
  <tr>
    <td>CNI</td>
    <td>0.5.2</td>
  </tr>
  <tr>
    <td>Stemcell</td>
    <td>3586.25</td>
  </tr>
  </tbody>
</table>

## v0.18.0

[Download](https://github.com/cloudfoundry-incubator/kubo-deployment/releases/download/v0.18.0/kubo-deployment-0.18.0.tgz) the release artifact.

**Release Date:** July 5, 2018

* **Deprecation notice** Removal of ABAC as an authorization mode option [#157695924](https://www.pivotaltracker.com/story/show/157695924)

* Improved the security of internal communication between kubernetes components. Stories: [#158357421](https://www.pivotaltracker.com/story/show/158357421) [#158356113](https://www.pivotaltracker.com/story/show/158356113) [#158356114](https://www.pivotaltracker.com/story/show/158356114)

* Allow storage classes to be configured with
 vSphere storage policies. [#157477844](https://www.pivotaltracker.com/story/show/157477844)
* Ensure nodes will recover correctly with vSphere HA [#158142038](https://www.pivotaltracker.com/story/show/158142038) [#158180949](https://www.pivotaltracker.com/story/show/158180949)

### Other minor changes

* Improved our tooling to deploy CFCR on BOSH-lite. [readme](https://github.com/cloudfoundry-incubator/kubo-deployment#deploy-cfcr-on-bosh-lite)
* Changed the set_kubeconfig input parameters. Note: credhub login required before running set_kubeconfig. [story](https://www.pivotaltracker.com/story/show/157841437)


### Component Versions

The following table lists the component versions for CFCR v0.18.0:

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
    <td>1.10.4</td>
  </tr>
  <tr>
    <td>Flannel</td>
    <td>0.10.0</td>
  </tr>
   <tr>
    <td>ETCD</td>
     <td>3.3.1</td>
  </tr>
  <tr>
    <td>Docker</td>
    <td>17.12.1-ce</td>
  </tr>
  <tr>
    <td>CNI</td>
    <td>0.5.2</td>
  </tr>
  <tr>
    <td>Stemcell</td>
    <td>3586.24</td>
  </tr>
  </tbody>
</table>

## v0.17.0

[Download](https://github.com/cloudfoundry-incubator/kubo-deployment/releases/download/v0.17.0/kubo-deployment-0.17.0.tgz) the release artifact.

**Release Date:** May 30, 2018

* Hardening and validation of running multiple Kubernetes Master Nodes across Availability Zones. The BOSH native manifest will deploy with three masters by default, our script based deployment will continue to deploy one. -- [epic](https://www.pivotaltracker.com/epic/show/3887236)

* Upgraded to Kubernetes v1.10 -- [story](https://www.pivotaltracker.com/story/show/156501233)

* Added smoke-tests to verify a deployed cluster -- [story](https://www.pivotaltracker.com/story/show/144108357), [readme](https://github.com/cloudfoundry-incubator/kubo-release/tree/master/src/smoke-tests)

* Allow operators to control feature gates on Kubernetes components. -- [PR](https://github.com/cloudfoundry-incubator/kubo-release/pull/201)

### Other changes

* Enable kube-controller-manager to issue certificates PR:[1](https://github.com/cloudfoundry-incubator/kubo-release/pull/199),[2](https://github.com/cloudfoundry-incubator/kubo-release/pull/187),[3](https://github.com/cloudfoundry-incubator/kubo-deployment/pull/281)
* Support AWS LoadBalancers when externalTrafficPolicy is set to Local -- [story](https://www.pivotaltracker.com/story/show/157044566)
* Add BOSH instance IDs to the node labels -- [PR](https://github.com/cloudfoundry-incubator/kubo-release/pull/190)
* Re-enable rpcbind to allow support for NFS -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/156686399)
* Remove unused terraform variable from kubo-lbs.tf -- [PR](https://github.com/cloudfoundry-incubator/kubo-deployment/pull/283)
* Operator rename.yml to set deployment name and etcd certificate alter -- [PR](https://github.com/cloudfoundry-incubator/kubo-deployment/pull/299)
* Drain script improvements -- [story](https://www.pivotaltracker.com/story/show/156999207)
* Enabled the automated vSphere cloud provider ([ref](https://vmware.github.io/vsphere-storage-for-kubernetes/documentation/existing.html#enabling-vsphere-cloud-provider)), and remove references to the cloud provider in vSphere deployed workers. -- stories:[1](https://www.pivotaltracker.com/story/show/157432912),[2](https://www.pivotaltracker.com/story/show/157735891)

### Announcement
In version v0.18 we are going to remove support for ABAC as a authorization mode.

### Component Versions

The following table lists the component versions for CFCR v0.17.0:

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
    <td>1.10.3</td>
  </tr>
  <tr>
    <td>Flannel</td>
    <td>0.10.0</td>
  </tr>
   <tr>
    <td>ETCD</td>
     <td>3.3.1</td>
  </tr>
  <tr>
    <td>Docker</td>
    <td>17.12.1-ce</td>
  </tr>
  <tr>
    <td>CNI</td>
    <td>0.5.2</td>
  </tr>
  <tr>
    <td>Stemcell</td>
    <td>3586.16</td>
  </tr>
  </tbody>
</table>

## v0.16.0

[Download](https://github.com/cloudfoundry-incubator/kubo-deployment/releases/download/v0.16.0/kubo-deployment-0.16.0.tgz) the release artifact.

**Release Date:** April 06, 2018

* A new etcd bosh release is being used (cfcr-etcd-release), this is to enable deployments with multiple etcd nodes -- [story](https://www.pivotaltracker.com/story/show/155114966).
* Prevent unnecessary route creation in default kube-controller-manager config -- [story](https://www.pivotaltracker.com/story/show/155977766).
* System specs can be applied even when a cloud provider is not configured -- [story](https://www.pivotaltracker.com/story/show/155936152).
* Enable inter-container communication in flannel, while retaining the original source ip.  -- [story](https://www.pivotaltracker.com/story/show/154384422).
* CFCR can be deployed to BOSH-lite  -- [story](https://www.pivotaltracker.com/story/show/154589411).

* **Fix:** Made the node drain (even) more robust -- [story](https://www.pivotaltracker.com/story/show/156164242).

### Other minor changes

- [Explicitly unmount overlayfs (#111)](https://github.com/cloudfoundry-incubator/kubo-release/pull/111)

### Component Versions

The following table lists the component versions for CFCR v0.16.0:

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
    <td>1.9.6</td>
  </tr>
  <tr>
    <td>Flannel</td>
    <td>0.10.0</td>
  </tr>
   <tr>
    <td>ETCD</td>
     <td>3.3.1</td>
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
    <td>3541.10</td>
  </tr>
  </tbody>
  </table>

### Conformance Tests Results

* [GCP](https://storage.googleapis.com/conformance-results/conformance-results-gcp-0.14.1-dev.55.tar.gz)
* [AWS](https://storage.googleapis.com/conformance-results/conformance-results-aws-0.14.1-dev.55.tar.gz)
* [vSphere](https://storage.googleapis.com/conformance-results/conformance-results-vsphere-0.14.1-dev.55.tar.gz)
* [OpenStack](https://storage.googleapis.com/conformance-results/conformance-results-openstack-0.14.1-dev.55.tar.gz)

## v0.15.0

[Download](https://github.com/cloudfoundry-incubator/kubo-deployment/releases/download/v0.15.0/kubo-deployment-0.15.0.tgz) the release artifact.

**Release Date:** March 20, 2018

* Removed edge case where timeouts during upgrades could lead to Etcd data loss -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/155814661).
* Added the ability to configure the HTTP(s) proxy to be used by the Kubernetes control plane -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/154717574).
* Made Kube-DNS use its own configuration so that the Kubelet configuration is not exposed in the Kube-DNS container -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/154873779).
- **Fix:** Allowed `\` and `|` to be used in vCenter passwords -- [cloudfoundry-incubator/kubo-release#180](https://github.com/cloudfoundry-incubator/kubo-release/issues/180).
* **Fix:** Made the node drain more robust -- [story #155549518](https://www.pivotaltracker.com/n/projects/2093412/stories/155549518), [story #156008895](https://www.pivotaltracker.com/n/projects/2093412/stories/156008895) and [cloudfoundry-incubator/kubo-release#181](https://github.com/cloudfoundry-incubator/kubo-release/issues/181).
* **Fix:** Included a kube-proxy dependency (conntrack) to fix error logs -- [story](https://www.pivotaltracker.com/story/show/154818002).
* **Fix:** Can use `kubectl top node` against a CFCR cluster. __Caveat:__ the `--heapster-scheme='https'` flag needs to be included -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/154906631).
* **GCP:** Stopped the Kube-Controller-Manager from creating unnecessary routes -- [story](https://www.pivotaltracker.com/story/show/155977766).
    - This was causing high Google Cloud API usage.
* **AWS:** Added the ability to provide AWS credentials in the BOSH manifest -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/155507606).
    - Previously, AWS access could only be granted by setting the IAM profile in the cloud config. In cases of BOSH Directors that are used by multiple deployments, it is necessary to provide the credentials in the BOSH manifest.

### Other minor changes

- [cloudfoundry-incubator/kubo-release#178](https://github.com/cloudfoundry-incubator/kubo-release/pull/178)
- [cloudfoundry-incubator/kubo-release#184](https://github.com/cloudfoundry-incubator/kubo-release/pull/184)
- [cloudfoundry-incubator/kubo-deployment#266](https://github.com/cloudfoundry-incubator/kubo-deployment/pull/266)
- [cloudfoundry-incubator/kubo-deployment#276](https://github.com/cloudfoundry-incubator/kubo-deployment/pull/276)
- [cloudfoundry-incubator/kubo-deployment#271](https://github.com/cloudfoundry-incubator/kubo-deployment/issues/271)

### Component Versions

The following table lists the component versions for CFCR v0.15.0:

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
    <td>1.9.5</td>
  </tr>
  <tr>
    <td>Flannel</td>
    <td>0.10.0</td>
  </tr>
   <tr>
    <td>ETCD</td>
     <td>3.3.2</td>
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
    <td>3541.9</td>
  </tr>
  </tbody>
  </table>

### Conformance Tests Results

* [GCP](https://storage.googleapis.com/conformance-results/conformance-results-gcp-0.14.1-dev.55.tar.gz)
* [AWS](https://storage.googleapis.com/conformance-results/conformance-results-aws-0.14.1-dev.55.tar.gz)
* [vSphere](https://storage.googleapis.com/conformance-results/conformance-results-vsphere-0.14.1-dev.55.tar.gz)
* [OpenStack](https://storage.googleapis.com/conformance-results/conformance-results-openstack-0.14.1-dev.55.tar.gz)

## v0.14.0

**Release Date:** February 20, 2018

* Kubernetes v1.9.3 -- [cloudfoundry-incubator/kubo-release#176](https://github.com/cloudfoundry-incubator/kubo-release/pull/176).
* Flannel v0.10.0 -- [cloudfoundry-incubator/kubo-release#169](https://github.com/cloudfoundry-incubator/kubo-release/pull/169).
* BOSH DNS v0.2.0 -- [cloudfoundry-incubator/kubo-deployment#261](https://github.com/cloudfoundry-incubator/kubo-deployment/pull/261).
* GOVC v0.16.0.
* Golang v1.9.4.
* BOSH Stemcell v3541.4.
* CFCR can now be deployed on an environment paved by [BBL](https://github.com/cloudfoundry/bosh-bootloader) -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/154592571).
* Exposed OpenID authentication properties -- [cloudfoundry-incubator/kubo-release#101](https://github.com/cloudfoundry-incubator/kubo-release/pull/101).
* `logging-level` BOSH property can be used to control the logging level of kube-proxy -- [cloudfoundry-incubator/kubo-release#163](https://github.com/cloudfoundry-incubator/kubo-release/pull/163).
* HTTP(s) Proxy BOSH properties will be used for Kubernetes interactions with the IaaS -- [cloudfoundry-incubator/kubo-release#130](https://github.com/cloudfoundry-incubator/kubo-release/pull/130).
* Nodes can now be deployed across multiple AZs on GCP -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/154473040).
  - Nodes get tagged appropriately by Kubernetes to ensure that workloads are properly spread across AZs.
* System workloads are now applied as part of the `apply-addons` BOSH errand -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/154838510).
  - System workloads have been a cause of many deployment issues.
* Enabled the API server audit logs -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/154051305).
  - Audit logs can be disabled if the `kube-apiserver.enable_audit_logs` BOSH property is set to `false`.
* Disabled the read-only port in the Kubelet -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/154536109).
* Disabled cAdvisor in Kubelet -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/154536108).
* Disabled the security context manipulation when privileged containers are off -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/154536111).
* The API server will not try to fix malformed requests anymore for security reasons -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/154727218).
* The API Server will clean up terminated pods more often to avoid running out of disk space -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/154727216).
* The API server will unmount volumes of terminated pods for security reasons -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/154727217).
* Most BOSH jobs switched to use [BPM](https://github.com/cloudfoundry-incubator/bpm-release) -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/153690804).
   - From the BPM readme: "[BPM] crucially provides a security barrier such that if one of the jobs on your machine is compromised then the incident is limited to just that job rather than all jobs on the same machine".
* **OpenStack:** Exposed `cloud-provider.openstack.ignore-volume-az` BOSH property for the OpenStack Cloud Provider -- [cloudfoundry-incubator/kubo-release#166](https://github.com/cloudfoundry-incubator/kubo-release/pull/166).
* **OpenStack:** Exposed `region` BOSH variable for the OpenStack Cloud Provider -- [cloudfoundry-incubator/kubo-deployment#262](https://github.com/cloudfoundry-incubator/kubo-deployment/issues/262).
* **Fix:** UAA credentials and vCenter passwords are now redacted in BOSH logs -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/154439227).
* **Fix:** to ensure that workers will pick the correct node name during rolling upgrades -- [cloudfoundry-incubator/kubo-release#170](https://github.com/cloudfoundry-incubator/kubo-release/pull/170).
* **Fix:** to ensure that nodes get properly drained before they stop, in order to minimize workload downtime during a rolling upgrade -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/155218338).
* **vSphere Fix:** vCenter password with special characters (`&`, `#`, etc) can now be used with CFCR without breaking the deployment -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/154922900).
* **Experimental:** An [ops-file](https://www.pivotaltracker.com/n/projects/2093412/stories/154472172) can now be used in conjunction to the `kubo-deployment` in order to experiment with the multi-master setup -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/154472172).

### Component Versions

The following table lists the component versions for CFCR v0.14.0:

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
    <td>1.9.3</td>
  </tr>
  <tr>
    <td>Flannel</td>
    <td>0.10.0</td>
  </tr>
   <tr>
    <td>ETCD</td>
     <td>3.2.14</td>
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
    <td>3541.4</td>
  </tr>
  </tbody>
  </table>

### Conformance Tests Results

* [GCP](https://storage.googleapis.com/conformance-results/conformance-results-gcp-0.14.0-dev.23.tar.gz)
* [AWS](https://storage.googleapis.com/conformance-results/conformance-results-aws-0.14.0-dev.23.tar.gz)
* [vSphere](https://storage.googleapis.com/conformance-results/conformance-results-vsphere-0.14.0-dev.23.tar.gz)
* [OpenStack](https://storage.googleapis.com/conformance-results/conformance-results-openstack-0.13.1-dev.8.tar.gz)

## v0.13.0

**Release Date:** January 25, 2018

[Download](https://github.com/cloudfoundry-incubator/kubo-deployment/releases/download/v0.13.0/kubo-deployment-0.13.0.tgz) the release artifact.

* Kubernetes 1.9.2 -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/154439228) and [story](https://www.pivotaltracker.com/n/projects/2093412/stories/153627570).
* Flannel 0.9.1 -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/154474110).
* RBAC as the default authorization mode.
* Support for VM power-offs and restarts -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/154554975).
    * Reliance on certain functionality provided by BOSH was causing restarting VMs to fail.
* Secure communications between system specs (Dashboard, Heapster and InfluxDB) -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/154328284) and [story](https://www.pivotaltracker.com/story/show/154328283).
* Ability to configure the timeout for system specs -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/154439229).
    * The BOSH property is `kubernetes-system-specs.timeout-sec` and is set to 20 minutes by default.
* Ability to update addon specs without experiencing API downtime -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/154312484).
* Ability to get diagnostic information if a system pod fails to be applied -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/154207050).
* Ability to have the default storage class be used in PVCs that do not specify a storage class -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/154536308).
* Ability to rotate the Kubernetes API certificate -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/154522561).
* Ability to use the syslog addon in a CFCR deployment -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/154357886).
* **Fix:** to not print secrets in user-facing scripts -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/153851093).
* **Fix** to not have more than one nodes go down during an upgrade -- [story](https://www.pivotaltracker.com/n/projects/2093412/stories/154051299).
* **vSphere** **Fix:** to avoid a synchronization issue that was causing master to fail to start -- [story](https://www.pivotaltracker.com/story/show/154413170).
* **OpenStack** **Fix** to have CFCR properly configure Kubernetes in order to communicate securely (TLS) to OpenStack -- [cloudfoundry-incubator/kubo-release#156](https://github.com/cloudfoundry-incubator/kubo-release/pull/156).

### Component Versions

The following table lists the component versions for CFCR v0.13.0:

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
    <td>1.9.2</td>
  </tr>
  <tr>
    <td>Flannel</td>
    <td>0.9.1</td>
  </tr>
   <tr>
    <td>ETCD</td>
     <td>3.2.14</td>
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
    <td>3468.20</td>
  </tr>
  </tbody>
  </table>

### Conformance Tests Results

* [GCP](https://storage.googleapis.com/conformance-results/conformance-results-gcp-0.12.1-dev.76.tar.gz)
* [AWS](https://storage.googleapis.com/conformance-results/conformance-results-aws-0.12.1-dev.76.tar.gz)
* [vSphere](https://storage.googleapis.com/conformance-results/conformance-results-vsphere-0.12.1-dev.76.tar.gz)
* [OpenStack](https://storage.googleapis.com/conformance-results/conformance-results-openstack-0.12.1-dev.76.tar.gz)

## v0.12.0

**Release Date:** January 10, 2018

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

The following table lists the component versions for CFCR v0.12.0:

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
4. See the [BOSH Bootloader repository](https://github.com/cloudfoundry/bosh-bootloader) for information on how to update the BOSH Director.
5. See [CFCR repository](https://github.com/cloudfoundry-incubator/kubo-release) for information about upgrading your CFCR cluster.


1. Deploy the CFCR BOSH release. See the [CFCR repository](https://github.com/cloudfoundry-incubator/kubo-release) for instructions.


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
