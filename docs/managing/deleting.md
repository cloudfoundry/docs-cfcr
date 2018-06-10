#Deleting CFCR

This topic describes how to delete your Cloud Foundry Container Runtime (CFCR) deployment and destroy your BOSH environment.

!!! note
	CFCR was formerly known as Kubo, and many CFCR assets described in this topic still use the Kubo name.

##Delete CFCR

To delete your CFCR deployment, perform the following steps:

1. Log in to your BOSH Director with `bosh-cli -e BOSH-ENV log-in`, where `BOSH-ENV` is the name of your BOSH environment. For example:
<p class="terminal">$ bosh-cli -e my-bosh log-in
User (): admin
Password ():</p>
When prompted, enter `admin` for the user name and the admin password located in `KUBO_ENV/creds.yml`.
1. Delete your CFCR deployment. Run the following command:<br>
	`bosh-cli -e BOSH-ENV -d KUBO-DEPLOYMENT delete-deployment`

	Where:

	* `BOSH-ENV`: This is the name of your BOSH environment.
	* `KUBO-DEPLOYMENT`: This is the name of your CFCR deployment. To list your deployments, run `bosh-cli -e BOSH-ENV deployments`.

	For example:
	<p class="terminal">$ bosh-cli -e my-bosh -d my-kubo delete-deployment</p>

##Destroy BOSH Environment

To destroy all of the resources you created when deploying CFCR, perform the procedures below for your IaaS.

!!! note
	This section is still under development.

###Google Cloud Platform

Perform the following steps to destroy your BOSH environment on Google Cloud Platform (GCP):

1. Log in to your bastion VM from the Google Cloud Shell. Follow the steps in the [(Optional) Step 1: Set Up Your Environment](../installing/gcp/routing-gcp/#optional-step-1-set-up-your-environment) section of the <em>Configuring IaaS Routing for GCP</em> topic.
1. Change into the root of the `kubo-deployment` repo. Enter the following command:
	<p class="terminal">$ cd /share/kubo-deployment</p>
1. Execute the `destroy_bosh` script to destroy all of the VMs in your BOSH environment. Run the following command:
	`./bin/destroy_bosh KUBO_ENV SERVICE_ACCOUNT_KEY` 

	Where:

	* `KUBO_ENV` is the directory that contains your CFCR configuration. 
	* `SERVICE_ACCOUNT_KEY` is your GCP service account key. If you followed the procedures in [Deploying BOSH for CFCR on GCP](../installing/gcp/deploying-bosh-gcp/), this is `~/terraform.key.json`.

	For example:
	<p class="terminal">$ ./bin/destroy_bosh ~/kubo-env/kubo ~/terraform.key.json</p>

1. Change into the directory that contains the GCP Terraform templates. This directory must also contain your `terraform.tfstate` file. Enter the following command:
	<p class="terminal">$ cd ~/kubo-deployment/docs/terraform/gcp/platform</p>
1. Use Terraform to destroy the resources created by infrastructure paving. You must have set the same environment variables as in [Deploying BOSH for CFCR on GCP](../installing/gcp/deploying-bosh-gcp/). Enter the following command:
	<p class="terminal">$ docker run -i -t \
	    -e CHECKPOINT_DISABLE=1 \
	    -e "GOOGLE_CREDENTIALS=\${GOOGLE_CREDENTIALS}" \
	    -v \$(pwd):/\$(basename \$(pwd)) \
	    -w /\$(basename \$(pwd)) \
	    hashicorp/terraform:light destroy \
	    -var service_account_email=\${service_account_email} \
	    -var projectid=\${project_id} \
	    -var network=\${network} \
	    -var region=\${region} \
	    -var prefix=\${prefix:-cfcr} \
	    -var zone=\${zone} \
	    -var subnet_ip_prefix=\${subnet_ip_prefix} \
	    -state /$(basename \$(pwd))/${prefix:-cfcr}.tfstate</p>

1. Remove the Identity and Access Management (IAM) policy binding. Enter the following command:
	<p class="terminal">$ gcloud projects remove-iam-policy-binding \${project_id} \
      --member serviceAccount:\${service_account_email} \
      --role roles/owner</p>

1. Remove the service account. Enter the following command:
   <p class="terminal">$ gcloud iam service-accounts delete ${service_account_email}</p>
   
1. Remove the service account key JSON from the bastion VM. Enter the following command:
	<p class="terminal">$ rm ~/${prefix:-cfcr}-tf.key.json</p>
