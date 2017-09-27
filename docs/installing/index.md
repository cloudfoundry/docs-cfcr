#Installing and Configuring

This section describes how to install and configure Kubo on your cloud platform.

You can install Kubo on Google Cloud Platform (GCP), vSphere, Amazon Web Services (AWS), and OpenStack.

To install Kubo, follow the procedures in the topics below.

!!! tip
	If you encounter problems when installing Kubo, see the [Troubleshooting Kubo](../managing/troubleshooting.md) topic.

##Step 1: Prepare Your IaaS

Follow the procedures below for your IaaS. You must deploy BOSH for Kubo and configure routing before deploying Kubo.

* [Preparing GCP for Kubo](gcp/)
* [Preparing vSphere for Kubo](vsphere/)
* [Preparing AWS for Kubo](aws/)
* [Preparing OpenStack for Kubo](openstack/)

##Step 2: Deploy  

* [Deploying Kubo](deploying-kubo/)

##Deployment Options

* [Customizing Kubo](customizing-kubo/): Deploy a customized Kubo installation.
* [Configuring Cloud Foundry Routing](cf-routing/): Use Cloud Foundry to handle routing for Kubo instead of IaaS load balancers or HAProxy.