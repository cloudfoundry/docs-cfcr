#Preparing GCP for CFCR

This topic describes how to prepare Google Cloud Platform (GCP) for Cloud Foundry Container Runtime (CFCR).

##Prerequisites

You must have the following to install CFCR on GCP:

1. An existing GCP project.

1. A user account with the Owner role in the GCP project. For more information, see the [Understanding Roles](https://cloud.google.com/iam/docs/understanding-roles) topic in the GCP documentation.

1. You must enable the following API libraries:

	* Google Cloud Resource Manager API
	* Google Cloud Resource Manager API V2 (both the original and V2 versions are required)
	* Google Identity and Access Management (IAM) API

	To enable these APIs, perform the following steps:
    
	1. Navigate to the GCP Console.
	1. From the left-hand navigation, select **APIs & services > Library**.
	1. Search for each library listed above. If it is disabled, click the **Enable** button to enable it.

##Step 1: Pave your Infrastructure

Pave your infrastructure by following the procedures in the [Paving infrastructure on GCP](paving-infrastructure-gcp/) topic.

##Step 2: Configure Routing

Configure your load balancers for CFCR by following the procedures in the [Configuring IaaS Routing for GCP](routing-gcp/) topic.

If you want to use Cloud Foundry for routing instead of IaaS load balancers, see the [Configuring Cloud Foundry Routing](../cf-routing/) topic.

##Step 3: Deploy BOSH for CFCR on GCP

Pave your infrastructure and deploy the BOSH Director for CFCR by following the procedures in the [Deploying BOSH for CFCR on GCP](deploying-bosh-gcp/) topic.

##Step 4: Deploy CFCR

After deploying BOSH for CFCR and configuring your load balancers, continue to the [Deploying CFCR](../deploying-cfcr/) topic to finish the CFCR installation.

