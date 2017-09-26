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

[Deploying Kubo](deploying-kubo/) describes how to deploy a standard Kubo installation. If you want to deploy a customized Kubo installation, see [Customizing Kubo](customizing-kubo/).