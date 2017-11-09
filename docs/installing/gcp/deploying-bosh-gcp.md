#Deploying BOSH for CFCR on GCP

This topic describes how to deploy BOSH for Cloud Foundry Container Runtime (CFCR) on Google Cloud Platform (GCP). Installing CFCR requires deploying a BOSH Director. 

In the procedures below, you use [Terraform](https://www.terraform.io/docs/) to pave your infrastructure and create a bastion VM. Then you deploy a BOSH Director from the bastion VM. 

!!! note
	CFCR was formerly known as Kubo, and many CFCR assets described in this topic still use the Kubo name.

##Step 1: Set Up Your Shell Environment

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

	!!!note
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
	<p class="terminal">$ export service_account_email=\${prefix}terraform@\${project_id}.iam.gserviceaccount.com</p>
1. Configure `gcloud` to use your zone. Enter the following command:
	<p class="terminal">$ gcloud config set compute/zone ${zone}</p>
1. Configure `gcloud` to use your region. Enter the following command:
	<p class="terminal">$ gcloud config set compute/region ${region}</p>

##Step 2: Set Up a GCP Account for Terraform

Perform the following steps to set up a GCP account for Terraform:

1. From the Google Cloud Shell, create a service account for Terraform. Enter the following command:
	<p class="terminal">$ gcloud iam service-accounts create ${prefix}terraform</p>
1. Create a service account key. Enter the following command:
	<p class="terminal">$ gcloud iam service-accounts keys create ~/terraform.key.json \
    --iam-account ${service_account_email}</p>
1. Grant the new service account owner access to your project. Enter the following command:
	<p class="terminal">$ gcloud projects add-iam-policy-binding \${project_id} \
	  --member serviceAccount:${service_account_email} \
	  --role roles/owner</p>
1. Set your service account key as an environment variable. Enter the following command:
	<p class="terminal">$ export GOOGLE_CREDENTIALS=$(cat ~/terraform.key.json)</p>

	If Terraform refuses to accept the JSON key as the content of `GOOGLE_CREDENTIALS`, provide the path to the file instead, using `GOOGLE_APPLICATION_CREDENTIALS`. Enter the following command:
		<p class="terminal">$ export GOOGLE_APPLICATION_CREDENTIALS=~/terraform.key.json</p>

##Step 3: Deploy Bastion VM

Perform the following steps to deploy a bastion VM with a set of firewall rules that secures access to the CFCR deployment:

1. From the Google Cloud Shell, change into the home directory. Enter the following command:
	<p class="terminal">$ cd ~</p>
1. Get the latest version of `kubo-deployment`. Enter the following command:
	<p class="terminal">$ wget http<span>s://</span>storage.googleapis.com/kubo-public/kubo-deployment-latest.tgz</p>
1. Expand the tarball. Enter the following command:
	<p class="terminal">$ tar -xvf kubo-deployment-latest.tgz</p>
1. Change into the directory that contains the GCP Terraform templates. Enter the following command:
	<p class="terminal">$ cd ~/kubo-deployment/docs/user-guide/platforms/gcp</p>
1. Initialize the Terraform cloud provider. Enter the following command:
	<p class="terminal">$ docker run -i -t \
  -v \$(pwd):/\$(basename \$(pwd)) \
  -w /\$(basename $(pwd)) \
  hashicorp/terraform:light init</p>
1. Create the bastion and other resources. Enter the following command:
	<p class="terminal">$ docker run -i -t \
  -e CHECKPOINT_DISABLE=1 \
  -e "GOOGLE_CREDENTIALS=\${GOOGLE_CREDENTIALS}" \
  -v \$(pwd):/\$(basename \$(pwd)) \
  -w /\$(basename \$(pwd)) \
  hashicorp/terraform:light apply \
    -var service_account_email=\${service_account_email} \
    -var projectid=\${project_id} \
    -var network=\${network} \
    -var region=\${region} \
    -var prefix=\${prefix} \
    -var zone=\${zone} \
    -var subnet_ip_prefix=\${subnet_ip_prefix}
	</p>
	This command takes between 60 and 90 seconds to complete.

	!!! tip
		To preview the Terraform execution plan before applying it, run `plan` instead of `apply`.

1. Copy the service account key to the newly created bastion VM. Enter the following command:
	<p class="terminal">$ gcloud compute scp ~/terraform.key.json "${prefix}bosh-bastion":./ --zone ${zone}</p>
	If prompted to create SSH keys, enter `Y` and use an empty passphrase.

##Step 4: Generate BOSH Configuration

Perform the following steps to generate the configuration for your BOSH Director:

1. From the Google Cloud Shell, SSH onto the bastion VM. Enter the following command:
	<p class="terminal">$ gcloud compute ssh "${prefix}bosh-bastion" --zone ${zone}</p>
1. Change into the root of the `kubo-deployment` repo. Enter the following command:
	<p class="terminal">$ cd /share/kubo-deployment</p>
1. Set three environment variables with the following commands:
	<p class="terminal">$ export kubo_envs=~/kubo-env
$ export kubo_env_name=kubo
$ export kubo_env_path="\${kubo_envs}/\${kubo_env_name}"</p>

	!!! note
		`kubo_env_path` points to the directory containing the CFCR configuration. Later topics will refer to this path as `KUBO_ENV`.

1. Make a new directory path with the following command:
	<p class="terminal">$ mkdir -p "${kubo_envs}"</p>
1. Generate a CFCR configuration template. Enter the following command:
	<p class="terminal">$ ./bin/generate_env_config "${kubo_envs}" ${kubo_env_name} gcp</p>
1. Execute the `update_gcp_env` script to apply the default network settings configured during the infrastructure paving to the template. Enter the following command:
	<p class="terminal">$ /usr/bin/update_gcp_env "${kubo_env_path}/director.yml"</p>

	!!! tip
		You can directly edit the configuration file located at `${kubo_env_path}/director.yml`.

	!!! warning
		If you want to configure Cloud Foundry to handle routing for CFCR, **do not continue to the next section**. Perform the procedures in [Configuring Cloud Foundry Routing](../cf-routing/) before deploying the BOSH Director. 

##Step 5: Deploy BOSH Director

Perform the following steps to deploy a BOSH Director from the bastion VM:

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

If you plan to use IaaS routing for CFCR, continue to [Configure IaaS Routing for GCP](routing-gcp/).

If you have configured Cloud Foundry routing, continue to [Deploying CFCR](../deploying-cfcr/).