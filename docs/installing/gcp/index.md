#Preparing GCP for Kubo

This topic describes how to prepare Google Cloud Platform (GCP) for Kubo.

##Prerequisites

You must have the following to install Kubo on GCP:

1. An existing GCP project.

1. A user account with the Owner role in the GCP project. For more information, see the [Understanding Roles](https://cloud.google.com/iam/docs/understanding-roles) topic in the GCP documentation.

1. You must enable the Google Identity and Access Management (IAM) and Google Cloud Resource Manager APIs. To enable these APIs, perform the following steps:
	1. Navigate to the GCP Console.
	1. From the left-hand navigation, select **APIs & services > Dashboard**.
	1. Locate the **Google Cloud Resource Manager API** and the **Google Identity and Access Management API** entries. If they are disabled, click the **Enable** link to enable them.

##Step 1: Deploy BOSH for Kubo on GCP

Pave your infrastructure and deploy the BOSH Director for Kubo by following the procedures in the [Deploying BOSH for Kubo on GCP](deploying-bosh-gcp/) topic.

##Step 2: Configure IaaS Routing for GCP

Configure your load balancers for Kubo by following the procedures in the [Configuring IaaS Routing for GCP](routing-gcp/) topic.

##Step 3: Deploy Kubo

After deploying BOSH for Kubo and configuring your load balancers, continue to the [Deploying Kubo](../deploying-kubo/) topic to finish the Kubo installation.

