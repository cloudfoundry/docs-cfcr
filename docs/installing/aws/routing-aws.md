#Configuring IaaS Routing for AWS

This topic describes how to configure the Amazon Web Services (AWS) load balancers to handle routing for Cloud Foundry Container Runtime (CFCR).

Before completing the procedures in this topic, you must have performed the steps in [Deploying BOSH for CFCR on AWS](deploying-bosh-aws/). After finishing this topic, continue to [Deploying CFCR](../deploying-cfcr/).

If you want to use Cloud Foundry for routing instead of IaaS load balancers, see the [Configuring Cloud Foundry Routing](../cf-routing/) topic.

!!! note
	CFCR was formerly known as Kubo, and many CFCR assets described in this topic still use the Kubo name.

##Configure IaaS Routing

1. If you are not in the same shell session as you were when completing the procedures in [Deploying BOSH for CFCR on AWS](deploying-bosh-aws/), perform the following steps:
	1. Change into your Terraform working directory with the following command:
		<p class="terminal">$ cd ~/kubo-deployment/docs/terraform/aws/platform</p>
	1. SSH onto the bastion VM with the following command:
		<p class="terminal">$ ssh -i ~/deployer.pem ubuntu@$(terraform output bosh-bastion-ip)</p>
	1. Set the `kubo_env_name` environment variable with the following command:
		<p class="terminal">$ export kubo_env_name=kubo</p>
1. On the bastion VM, change into the directory that contains the AWS user guide. Enter the following command:
	<p class="terminal">$ cd /share/kubo-deployment/docs/terraform/aws/routing</p>
1. Set the path of the state directory as an environment variable named `state_dir`. Enter the following command:
	<p class="terminal">$ export state_dir=~/kubo-env/${kubo_env_name}</p> 
1. Set the path of the Terraform state file as an environment variable named `kubo_terraform_state`. Enter the following command:
	<p class="terminal">$ export kubo_terraform_state=${state_dir}/terraform.tfstate</p>
1. Set the access key ID and secret access key for an IAM user with the AdministratorAccess policy as environment variables named `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`. For example:
	<p class="terminal">$ export AWS_ACCESS_KEY_ID=AKIDZAAFAA0AOABCJ00Z
$ export AWS_SECRET_ACCESS_KEY=dsfSDFKOSDFKOasdmasdKSADOK</p>

	!!! note
		This is the same IAM user whose credentials you exported in the [Step 1: Set Up Your Shell Environment](deploying-bosh-aws/#step-1-set-up-your-shell-environment) section of the <em>Deploying BOSH for CFCR on AWS</em> topic. This is not the IAM user you created for BOSH in the [Step 4: Create IAM User](deploying-bosh-aws/#step-4-create-iam-user) section of the same topic. 

1. Initialize the Terraform working directory. Enter the following command:
	<p class="terminal">$ terraform init</p>
1. Create the resources with the following command:
	<p class="terminal">$ terraform apply \
   -var region=\${region} \
   -var vpc_id=\${vpc_id} \
   -var node_security_group_id=\${default_security_groups} \
   -var public_subnet_id=\${public_subnet_id} \
   -var prefix=\${prefix} \
   -state=\${kubo_terraform_state}</p>
1. Set the `master_target_pool` and `kubernetes_master_host` environment variables with the following commands:
	<p class="terminal">$ export master_target_pool=\$(terraform output -state=${kubo_terraform_state} cfcr_master_target_pool)
$ export kubernetes_master_host=\$(terraform output -state=${kubo_terraform_state} master_lb_ip_address) </p>
1. Update the CFCR environment by running the the following commands:
	<p class="terminal">$ /usr/bin/set_iaas_routing "${kubo_env_path}/director.yml"</p>

	!!! tip
		You can also set the configuration manually by editing `KUBO_ENV/director.yml`.

After configuring routing, continue to the [Deploying CFCR](../deploying-cfcr/) topic.
