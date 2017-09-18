#Preparing vSphere for Kubo

This topic describes how to prepare vSphere for Kubo.

##Step 1: Deploy BOSH for Kubo on vSphere

Configure the correct user privileges and deploy the BOSH Director for Kubo by following the procedures in the [Deploying BOSH for Kubo on vSphere](deploying-bosh-vsphere/) topic.

##Step 2: Configure HAProxy for vSphere

Configure HAProxy to handle routing for Kubo on vSphere by following the procedures in the [Configuring HAProxy for vSphere](routing-vsphere/) topic.

##Step 3: Deploy Kubo

After deploying BOSH for Kubo and configuring your load balancers, continue to the [Deploying Kubo](../deploying-kubo/) topic to finish the Kubo installation.

