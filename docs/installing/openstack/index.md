#Preparing OpenStack for CFCR

This topic describes how to prepare OpenStack for Cloud Foundry Container Runtime (CFCR).

##Prerequisites

You must have the following to install CFCR on OpenStack:

1. An OpenStack environment running one of the following supported releases:

    - *(Recommended)* [Mitaka](http://www.openstack.org/software/mitaka)

    	!!! note
    		The CFCR development team actively tests on Mitaka.
    
    - [Liberty](http://www.openstack.org/software/liberty)
    - [Newton](http://www.openstack.org/software/newton)

1. The following OpenStack services:
   
    - [Identity](https://www.openstack.org/software/project-navigator#security)
    - [Compute](https://www.openstack.org/software/project-navigator#compute)
    - [Image](https://www.openstack.org/software/project-navigator#compute)
    - *(Recommended)* [OpenStack Networking](https://www.openstack.org/software/project-navigator#networking), which provides network scaling and automated management functions

	!!! note
		Nova networking is known to work, but the CFCR team does not actively test it because it is deprecated.
		    
1. An existing OpenStack project
1. An OpenStack key pair
1. A network configured to allow the following:
    - Incoming TCP traffic on port 8443 for the Kubernetes API server 
    - Incoming TCP traffic for port range 30000—32765
    - Incoming UDP traffic for port 8285 within the CFCR network
    - Incoming TCP traffic for ports 8844 and 25555 for operators
    - Outgoing TCP traffic for port 53 within the CFCR network
   
    !!! tip
    	You can find the network settings in **Compute** > **Access & Security**, clicking the Manage Rules button for your security group.
    
	*(Optional)* If you plan to configure Cloud Foundry to handle routing for CFCR, the network must also allow the following:

    - Outgoing TCP traffic to Cloud Foundry routing API host and port
    - Outgoing TCP traffic to Cloud Foundry NATS hosts and port
    - Outgoing TCP traffic to Cloud Foundry router host and CFCR port

##Step 1: Deploy BOSH for CFCR on OpenStack

Deploy the BOSH Director for CFCR by following the procedures in the [Deploying BOSH for CFCR on OpenStack](deploying-bosh-openstack/) topic.

##Step 2: Deploy CFCR

After deploying BOSH for CFCR, continue to the [Deploying CFCR](../deploying-cfcr/) topic to finish the CFCR installation.
