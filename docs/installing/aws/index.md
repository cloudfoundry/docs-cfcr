#Preparing AWS for Kubo

This topic describes how to prepare Amazon Web Services (AWS) for Kubo.

##Prerequisites

You must have the following to install Kubo on AWS:

1. An existing AWS account
1. An access key ID and secret access key for an IAM user with the AdministratorAccess policy
1. A VPC in the zone where you want to deploy Kubo, with a CIDR range that has at least an `/22` netmask

	!!! warning
		Do not create this VPC by clicking **Start VPC Wizard**.

1. DNS hostnames enabled for the VPC 

	!!! tip
		To enable DNS hostnames for a VPC, select the VPC in the VPC Dashboard of the AWS Console and click **Actions > Edit DNS Hostnames**.

##Step 1: Deploy BOSH for Kubo on AWS

Pave your infrastructure and deploy the BOSH Director for Kubo by following the procedures in the [Deploying BOSH for Kubo on AWS](deploying-bosh-aws/) topic.

##Step 2: Configure Routing

Configure your load balancers for Kubo by following the procedures in the [Configuring IaaS Routing for AWS](routing-aws/) topic.

If you want to use Cloud Foundry for routing instead of IaaS load balancers, see the [Configuring Cloud Foundry Routing](../cf-routing/) topic.

##Step 3: Deploy Kubo

After deploying BOSH for Kubo and configuring your load balancers, continue to the [Deploying Kubo](../deploying-kubo/) topic to finish the Kubo installation.

