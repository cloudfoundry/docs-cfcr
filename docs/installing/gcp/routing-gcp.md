#Configuring IaaS Routing for GCP

This topic describes how to configure the Google Cloud Platform (GCP) load balancers to handle routing for Cloud Foundry Container Runtime (CFCR).

Before completing the procedures in this topic, you must have performed the steps in [Deploying BOSH for CFCR on GCP](deploying-bosh-gcp/). After finishing this topic, continue to [Deploying CFCR](../deploying-cfcr/).

If you want to use Cloud Foundry for routing instead of IaaS load balancers, see the [Configuring Cloud Foundry Routing](../cf-routing/) topic.

!!! note
	CFCR was formerly known as Kubo, and many CFCR assets described in this topic still use the Kubo name.

!!! tip
		Execute all of the commands in this topic from the Google Cloud Shell, not from your local machine.

##(Optional) Step 1: Set Up Your Environment

If you are still working in the same Google Cloud Shell session from the [Deploying BOSH for CFCR on GCP](deploying-bosh-gcp/) topic, skip to [Step 2: Configure Load Balancers](#step-2-configure-load-balancers).

If you have started a new Google Cloud Shell session, perform the following steps:

1. Set the prefix and zone from the [Step 1: Set Up Your Shell Environment](deploying-bosh-gcp/#step-1-set-up-your-shell-environment) section of the [Deploying BOSH for CFCR on GCP](https://docs-kubo.cfapps.io/installing/gcp/deploying-bosh-gcp/) topic as environment variables. For example:
	<p class="terminal">$ export prefix=my-kubo
$ export zone=us-west1-a</p>
1. SSH onto the bastion VM. Enter the following command:
	<p class="terminal">$ gcloud compute ssh "${prefix}-bosh-bastion" --zone ${zone}</p>
1. Set the `kubo_env_name` environment variable to `kubo`. Enter the following command:
	<p class="terminal">$ export kubo_env_name=kubo</p> 

##Step 2: Configure Load Balancers

1. Change into the GCP user guide directory of the `kubo-deployment` repo. Enter the following command:
	<p class="terminal">$ cd /share/kubo-deployment/docs/terraform/gcp/routing
</p>
1. Export your state information as environment variables with the following commands:
	<p class="terminal">$ export state_dir=~/kubo-env/${kubo_env_name}
$ export kubo_terraform_state=${state_dir}/terraform.tfstate</p>
1. Use Terraform to create the GCP resources for CFCR:
	<p class="terminal">$ terraform apply \
    -var network=\${network} \
    -var projectid=\${project_id} \
    -var region=\${region} \
    -var prefix=\${prefix:-cfcr} \
    -var ip_cidr_range="\${subnet_ip_prefix}.0/24" \
    -state=\${kubo_terraform_state}</p>
1. Set the master target pool from the outputted Terraform state file as an environment variable. Enter the following command:
	<p class="terminal">$ export master_target_pool=\$(terraform output -state=\${kubo_terraform_state} cfcr_master_target_pool)</p>
1. Set the Kubernetes master host from the outputted Terraform state file as an environment variable. Enter the following command:
	<p class="terminal">$ export kubernetes_master_host=\$(terraform output -state=\${kubo_terraform_state} master_lb_ip_address)</p>
1. Update the CFCR environment by running the `set_iaas_routing` script. Enter the following command:
	<p class="terminal">$ /usr/bin/set_iaas_routing "${state_dir}/director.yml"</p>

	!!! tip
		You can also set the configuration manually by editing `KUBO_ENV/director.yml`.

After configuring routing, continue to the [Deploying CFCR](../deploying-cfcr/) topic.
