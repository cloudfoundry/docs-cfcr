#Deploying BOSH for CFCR on GCP

This topic describes how to deploy BOSH for Cloud Foundry Container Runtime (CFCR) on Google Cloud Platform (GCP). Installing CFCR requires deploying a BOSH Director. 

In the procedures below, you deploy a BOSH Director from the bastion VM you created in [Paving Infrastructure on GCP](paving-infrastructure-gcp/).

Before completing the procedures in this topic, you must have performed the steps in [Configuring IaaS Routing for GCP](routing-gcp/). After finishing this topic, continue to [Deploying CFCR](../deploying-cfcr/).

##(Optional) Step 1: Set Up Your Environment

If you are still working in the same Google Cloud Shell session from the [Paving Infrastructure on GCP](paving-infrastructure-gcp/) topic, skip to [Step 2: Deploy BOSH Director](#step-2-deploy-bosh-director).

If you have started a new Google Cloud Shell session, perform the following steps:

1. Set the prefix and zone from the [Step 1: Set Up Your Shell Environment](paving-infrastructure-gcp/#step-1-set-up-your-shell-environment) section of the [Paving Infrastructure on GCP](https://docs-kubo.cfapps.io/installing/gcp/paving-infrastructure-gcp/) topic as environment variables. For example:
	<p class="terminal">$ export prefix=my-kubo
$ export zone=us-west1-a
$ export network=kubo-network
$ export subnet_ip_prefix="10.0.1"</p>
1. SSH onto the bastion VM. Enter the following command:
	<p class="terminal">$ gcloud compute ssh "${prefix:-cfcr}-bosh-bastion" --zone ${zone}</p>
1. Set the `kubo_env_name` environment variable to `kubo`. Enter the following command:
	<p class="terminal">$ export kubo_env_name=kubo</p> 

##Step 2: Deploy BOSH Director

Perform the following steps to deploy a BOSH Director from the bastion VM:
1. Change into the root of the kubo-deployment repo. Enter the following command:
        <p class="terminal">$ cd /share/kubo-deployment</p>
1. Deploy the BOSH Director for CFCR. Enter the following command:
	<p class="terminal">$ ./bin/deploy_bosh "${kubo_env_path}" ~/terraform.key.json</p>

	The `deploy_bosh` script uses the CFCR configuration and the GCP key generated previously to deploy a BOSH Director with all of the necessary components to install CFCR. 

	After the script completes, `KUBO_ENV` contains the following:

	* Credentials and SSL certificates for the BOSH Director, stored in `creds.yml`

		!!! warning
			The `creds.yml` file contains sensitive information and should not be under version control.

	* The [deployment state](https://bosh.io/docs/cli-envs.html#deployment-state), stored in `state.json`

		!!! note
			Subsequent runs of `deploy_bosh` will use `creds.yml` and `state.json` to apply changes to the BOSH environment.

##Step 3: Access the BOSH Director


1. Set your environment to access the BOSH Director. Enter the following command:
  <p class="terminal">$ BOSH_ENV=${kubo_env_path} source /share/kubo-deployment/bin/set_bosh_environment</p>

1. Use [BOSH CLI v2](https://bosh.io/docs/cli-v2.html) commands to interact with the BOSH Director. For example, enter the following command:
  <p class="terminal">$ bosh environment</p>
  BOSH provides information about your environment. For example:
  <p class="terminal">Using environment '10.0.1.252' as client 'bosh_admin'<br>
Name      my-kubo-bosh
UUID      a7e779dd-f9cc-43f2-b491-7358556bc730
Version   264.1.0 (00000000)
CPI       google_cpi
Features  compiled_package_cache: disabled
          config_server: enabled
          dns: disabled
          snapshots: disabled
User      bosh_admin<br>
Succeeded</p>

After deploying the BOSH Director, continue to [Deploying CFCR](../deploying-cfcr/).
