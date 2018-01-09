#Installing and Configuring

This section describes how to install and configure Cloud Foundry Container Runtime (CFCR) on your cloud platform.

You can install CFCR on Google Cloud Platform (GCP), vSphere, Amazon Web Services (AWS), and OpenStack.

To install CFCR, follow the procedures in the topics below.

!!! tip
	If you encounter problems when installing CFCR, see the [Troubleshooting CFCR](../managing/troubleshooting.md) topic.

##Step 1: Prepare Your IaaS

Follow the procedures below for your IaaS. You must deploy BOSH for CFCR and configure routing before deploying CFCR.

* [Preparing GCP for CFCR](gcp/)
* [Preparing vSphere for CFCR](vsphere/)
* [Preparing AWS for CFCR](aws/)
* [Preparing OpenStack for CFCR](openstack/)

##Step 2: Deploy  

* [Deploying CFCR](deploying-cfcr/)

##Deployment Options

* [Customizing CFCR](customizing-cfcr/): Deploy a customized CFCR installation.
* [Configuring Cloud Foundry Routing](cf-routing/): Use Cloud Foundry to handle routing for CFCR instead of IaaS load balancers.
* [Configuring Pluggable Add-ons](pluggable-addons/): Pluggable add-ons are Kubernetes workloads that start immediately after CFCR cluster deployment. They let you add custom services to a CFCR cluster.
