#Deploying BOSH for Kubo on GCP

This topic describes how to deploy BOSH for Kubo on Google Cloud Platform (GCP). Installing Kubo requires deploying a BOSH Director. 

After completing the procedures in this topic, continue to the [Configure IaaS Routing for GCP](routing-gcp/) topic.

In the procedures below, you use [Terraform](https://www.terraform.io/docs/) to pave your infrastructure and create a bastion VM. Then you deploy a BOSH Director from the bastion VM. 

##Step 1: Set Up Your Shell Environment

Perform the following steps to set up your Google Cloud Shell environment: 

1. Log in to the GCP Console.
1. From the left-hand navigation, click **VPC network**.
1. Click **Create VPC Network**.
1. Enter a name for your network, such as `kubo-network`, and click **Create**. 
1. Open the Google Cloud Shell by clicking the `>_` icon in the upper right of the GCP Console.

	!!!note
		Execute all of the commands in this topic from the Google Cloud Shell, not from your local machine.

1. If you are deploying Kubo more than once, you must set a unique prefix for every installation. Export an environment variable in the Google Cloud Shell to set a prefix for your Kubo installation. Use letters and dashes only. For example:
	<p class="terminal">$ export prefix=my-kubo</p>
1. Set the name of your GCP network as an environment variable. For example:
	<p class="terminal">$ export network=kubo-network</p>
1. Set your subnet prefix as an environment variable. You must configure a CIDR range with a mask that is 24 bits long. A CIDR range of `10.0.1.0/24` would result in a subnet prefix of `10.0.1`. For example:
	<p class="terminal">$ export subnet_ip_prefix="10.0.1"</p>
1. Set your project ID as an environment variable using `gcloud`. Enter the following command:
	<p class="terminal">$ export project_id=$(gcloud config get-value project)</p>
1. Set your region where you want to deploy Kubo as an environment variable. For example:
	<p class="terminal">$ export region=us-west1</p>
1. Set your zone where you want to deploy Kubo as an environment variable. For example:
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

##Step 3: Deploy Bastion VM

Perform the following steps to deploy a bastion VM with a set of firewall rules that secures access to the Kubo deployment:

1. From the Google Cloud Shell, change into the user directory. Enter the following command:
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
	If prompted to create SSH keys, enter `Y`.

##Step 4: Deploy BOSH Director

Perform the following steps to deploy a BOSH Director from the bastion VM:

1. From the Google Cloud Shell, SSH onto the bastion VM. Enter the following command:
	<p class="terminal">$ gcloud compute ssh "${prefix}bosh-bastion" --zone ${zone}</p>
1. Change into the root of the `kubo-deployment` repo. Enter the following command:
	<p class="terminal">$ cd /share/kubo-deployment</p>
1. Set three Kubo environment variables with the following commands:
	<p class="terminal">$ export kubo_envs=~/kubo-env
$ export kubo_env_name=kubo
$ export kubo_env_path="\${kubo_envs}/\${kubo_env_name}"</p>

	!!! note
		`kubo_env_path` points to the directory containing the Kubo configuration. Later topics will refer to this path as `KUBO_ENV`.

1. Make a new directory path with the following command:
	<p class="terminal">$ mkdir -p "${kubo_envs}"</p>
1. Generate a Kubo configuration template. Enter the following command:
	<p class="terminal">$ ./bin/generate_env_config "${kubo_envs}" ${kubo_env_name} gcp</p>
1. Execute the `update_gcp_env` script to apply the default network settings configured during the infrastructure paving to the template. Enter the following command:
	<p class="terminal">$ /usr/bin/update_gcp_env "${kubo_env_path}/director.yml"</p>

	!!! tip
		You can directly edit the configuration file located at `${kubo_env_path}/director.yml`.

1. Deploy the BOSH Director for Kubo. Enter the following command:
	<p class="terminal">$ ./bin/deploy_bosh "${kubo_env_path}" ~/terraform.key.json</p>

	The `deploy_bosh` script deploys a BOSH Director with all of the necessary components to install Kubo. 

	After the script completes, `KUBO_ENV` contains the following:

	* Credentials and SSL certificates for the BOSH Director, stored in `creds.yml`

		!!! warning
			The `cred.yml` file contains sensitive information and should not be under version control.

	* The [deployment state](https://bosh.io/docs/cli-envs.html#deployment-state), stored in `state.json`

		!!! note
			Subsequent runs of `deploy_bosh` will use `creds.yml` and `state.json` to apply changes to the BOSH environment.

After deploying the BOSH Director, continue to the [Configure IaaS Routing for GCP](routing-gcp.html) topic.

