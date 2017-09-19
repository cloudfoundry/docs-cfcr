#Preparing AWS for Kubo

This topic describes how to prepare Amazon Web Services (AWS) for Kubo.

##Prerequisites

You must have the following to install Kubo on AWS:

1. An existing AWS account
1. A shell environment with the [Terraform CLI](https://www.terraform.io/intro/getting-started/install.html) installed
1. An access key ID and access key secret for an IAM user with administrator access
1. A VPC in the zone where you want to deploy Kubo, with a CIDR range that has at least an `/22` netmask

	!!! warning
		Do not create this VPC by clicking **Start VPC Wizard**.

1. DNS hostnames enabled for the VPC 

	!!! tip
		To enable DNS hostnames for a VPC, select the VPC in the VPC Dashboard of the AWS Console and click **Actions > Edit DNS Hostnames**.

##Step 1: Deploy BOSH for Kubo on AWS

Pave your infrastructure and deploy the BOSH Director for Kubo by following the procedures in the [Deploying BOSH for Kubo on AWS](deploying-bosh-aws/) topic.

##Step 2: Configure IaaS Routing for AWS

Configure your load balancers for Kubo by following the procedures in the [Configuring IaaS Routing for AWS](routing-aws/) topic.

##Step 3: Deploy Kubo

After deploying BOSH for Kubo and configuring your load balancers, continue to the [Deploying Kubo](../deploying-kubo/) topic to finish the Kubo installation.

