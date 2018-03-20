#Configuring IaaS Routing for GCP

This topic describes how to configure the Google Cloud Platform (GCP) load balancers to handle routing for Cloud Foundry Container Runtime (CFCR).

If you want to use Cloud Foundry for routing instead of IaaS load balancers, see the [Configuring Cloud Foundry Routing](../cf-routing/) topic.

!!! note
	CFCR was formerly known as Kubo, and many CFCR assets described in this topic still use the Kubo name.

!!! tip
		Execute all of the commands in this topic from the Google Cloud Shell, not from your local machine.

## Step 1: Set Up Your Shell Environment

Perform the following steps to set up your Google Cloud Shell environment:

1. Log in to the GCP Console.
1. From the left-hand navigation, click **VPC network**.
1. Click **Create VPC Network**.

    !!! tip
        If you plan to use [Cloud Foundry to handle routing](../cf-routing.html) for CFCR, do not create a new network. Instead, deploy CFCR in the same network as Cloud Foundry.

1. Enter a name for your network, such as `kubo-network`.
1. The form will force you to create a subnet for the VPC network. After the VPC has been created, you can delete this subnet.
1. Click **Create**.
1. Open the Google Cloud Shell by clicking the `>_` icon in the upper right of the GCP Console.

    !!! note
        Execute all of the commands in this topic from the Google Cloud Shell, not from your local machine.

1. If you are deploying CFCR more than once, you must set a unique prefix for every installation. Export an environment variable in the Google Cloud Shell to set a prefix for your CFCR installation. Use letters and dashes only. For example:
  <p class="terminal">$ export prefix=my-kubo</p>
1. Set the name of your GCP network as an environment variable. For example:
  <p class="terminal">$ export network=kubo-network</p>
1. Set your subnet prefix as an environment variable. You must configure a CIDR range with a mask that is 24 bits long. A CIDR range of `10.0.1.0/24` would result in a subnet prefix of `10.0.1`. For example:
  <p class="terminal">$ export subnet_ip_prefix="10.0.1"</p>
1. Set your project ID as an environment variable using `gcloud`. Enter the following command:
  <p class="terminal">$ export project_id=$(gcloud config get-value project)</p>
1. Set your region where you want to deploy CFCR as an environment variable. For example:
  <p class="terminal">$ export region=us-west1</p>
1. Set your zone where you want to deploy CFCR as an environment variable. For example:
  <p class="terminal">$ export zone=us-west1-a</p>
1. Set your service account email address as an environment variable. Enter the following command:
  <p class="terminal">$ export service_account_email=\${prefix:-cfcr}-terraform@\${project_id}.iam.gserviceaccount.com</p>
1. Configure `gcloud` to use your zone. Enter the following command:
  <p class="terminal">$ gcloud config set compute/zone ${zone}</p>
1. Configure `gcloud` to use your region. Enter the following command:
  <p class="terminal">$ gcloud config set compute/region ${region}</p>

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

After configuring routing, continue to the [Deploying BOSH for CFCR on GCP](deploying-bosh-gcp/) topic.
