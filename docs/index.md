#

![CFCR Logo](./images/common/cfcr-full.png)

This is the documentation for Cloud Foundry Container Runtime (CFCR), an open-source project that provides a solution for deploying and managing [Kubernetes](https://kubernetes.io/docs/home/) clusters using [BOSH](https://bosh.io/docs).

CFCR uses BOSH to improve the Kubernetes experience for both operators and developers. By leveraging BOSH, CFCR adds capabilities such as high availability, scaling, and self-healing to Kubernetes clusters.

CFCR was formerly known as Kubo, and many CFCR assets still use the Kubo name. CFCR does not require Cloud Foundry, but you can optionally configure a Cloud Foundry deployment to handle routing for CFCR.

See [What is CFCR?](./overview/what-is-cfcr/) to learn more.

## Installing and Configuring

**DEPRECATED**
>We no longer support the following strategies to deploy Bosh and CFCR. Please see [Bosh Bootloader](https://github.com/cloudfoundry/bosh-bootloader) on how to deploy Bosh and [CFCR Repository](https://github.com/cloudfoundry-incubator/kubo-release) to deploy CFCR.

The following topics describe how to install and configure CFCR on your cloud platform.

[Home](./installing/)

[Preparing GCP for CFCR](./installing/gcp/)  
[Preparing BOSH for CFCR on GCP](./installing/gcp/deploying-bosh-gcp/)  
[Configuring IaaS Routing for GCP](./installing/gcp/routing-gcp/)  

[Preparing vSphere for CFCR](./installing/vsphere/)  
[Deploying BOSH for CFCR on vSphere](./installing/vsphere/deploying-bosh-vsphere/)

[Preparing AWS for CFCR](./installing/aws/)  
[Deploying BOSH for CFCR on AWS](./installing/aws/deploying-bosh-aws/)  
[Configuring IaaS Routing for AWS](./installing/aws/routing-aws/)  

[Preparing OpenStack for CFCR](./installing/openstack/)  
[Deploying BOSH for CFCR on Openstack](./installing/openstack/)  

[Deploying CFCR](./installing/deploying-cfcr/)  
[Customizing CFCR](./installing/customizing-cfcr/)  
[Configuring Cloud Foundry Routing](./installing/cf-routing/)   
[Configuring Pluggable Add-ons](./installing/pluggable-addons/)

## Managing and Troubleshooting

The following topics describe how to manage and troubleshoot CFCR.

[Home](./managing/)  
[Using Docker Registries](./managing/using-docker/)  
[Deploying Tested Workloads](./managing/tested-workloads/)  
[Troubleshooting CFCR](./managing/troubleshooting/)  
[Deleting CFCR](./managing/deleting/)

## Certifications

<img alt="Kubernetes 1.10 Certification logo" src="https://github.com/cncf/artwork/blob/master/kubernetes/certified-kubernetes/1.10/color/certified-kubernetes-1.10-color.png" width="198px">

Kubernetes® is a registered trademark of The Linux Foundation in the United States and other countries, and is used pursuant to a license from The Linux Foundation.
