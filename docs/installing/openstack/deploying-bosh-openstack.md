#Deploying BOSH for CFCR on OpenStack

This topic describes how to deploy BOSH for Cloud Foundry Container Runtime (CFCR) on OpenStack. Installing CFCR requires deploying a BOSH Director. 

After completing the procedures in this topic, continue to the [Deploying CFCR](../deploying-cfcr/) topic. 

!!! note
	CFCR was formerly known as Kubo, and many CFCR assets described in this topic still use the Kubo name.

##Step 1: Generate a Configuration Template

Perform the following steps to generate a CFCR configuration template:

1. Ensure you are using a machine that has access to the VMs on the OpenStack network. Depending on your network topology, you may need to execute the commands below on a bastion host.
1. Change into the home directory. Enter the following command:
	<p class="terminal">$ cd ~</p>
1. Get the latest version of `kubo-deployment`. Enter the following command:
	<p class="terminal">$ wget http<span>s:/</span>/storage.googleapis.com/kubo-public/kubo-deployment-latest.tgz</p>
1. Expand the tarball. Enter the following command:
	<p class="terminal">$ tar -xvf kubo-deployment-latest.tgz</p>
1. Change into `kubo-deployment`. Enter the following command:
	<p class="terminal">$ cd ~/kubo-deployment</p>
1. Set three Kubo environment variables with the following commands:
	<p class="terminal">$ export kubo_env=~/kubo-env
$ export kubo_env_name=kubo
$ export kubo_env_path="\${kubo_env}/\${kubo_env_name}"</p>

    !!! note
		`kubo_env_path` points to the directory containing the CFCR configuration. Later topics will refer to this path as `KUBO_ENV`.

1. Make a new directory path with the following command:
	<p class="terminal">$ mkdir -p "${kubo_env}"</p>
1. Generate a CFCR configuration template:
   <p class="terminal">$ ./bin/generate_env_config "${kubo_env}" ${kubo_env_name} openstack</p>

##Step 2: Configure Routing

If you want to configure Cloud Foundry to handle routing for CFCR, perform the procedures in [Configuring Cloud Foundry Routing](../cf-routing/).

If you want to configure HAProxy to handle routing for CFCR, perform the procedures in [Configuring HAProxy](../haproxy/).

!!! note 
	OpenStack does not have first-party load-balancing support. You can configure HAProxy to handle external access to the Kubernetes master nodes for administration traffic, and the Kubernetes worker nodes for application traffic.

If you want to configure an external load balancer, perform the following steps:

1. Navigate to `KUBO_ENV` and open the newly created `director.yml` file.
1. Under the `IaaS routing mode settings` section, set the `routing_mode` to `external`.
1. Set the `kubernetes_master_host` to the IP address of your external load balancer.
1. Set the `kubernetes_master_port` to the port exposed by the external load balancer.
1. Comment out the `master_target_pool` property.

##Step 3: Deploy BOSH

Perform the following steps to deploy a BOSH Director:

1. Continue working in the `director.yml` file to configure BOSH by populating the remaining uncommented lines. For `default_key_name`, enter the name of an OpenStack key pair, and ensure you have a copy of the PEM-encoded private key file on your machine.
1. In the same directory, open the `director-secrets.yml` file.
1. Set the `openstack_password` to the OpenStack admin user password.

	!!! warning
		The `director-secrets.yml` file contains sensitive information and should not be under version control.

1. Deploy the BOSH Director for CFCR, supplying the path to the private key file as an argument. This is the private key whose name you set as `default_key_name` in `director.yml`. For example:

	<p class="terminal">$ ./bin/deploy_bosh "${kubo_env_path}" private_key.pem</p>

	The `deploy_bosh` script deploys a BOSH Director with all of the necessary components to install CFCR. 

	After the script completes, `KUBO_ENV` contains the following:

	* Credentials and SSL certificates for the BOSH Director, stored in `creds.yml`

		!!! warning
			The `cred.yml` file contains sensitive information and should not be under version control.

	* The [deployment state](https://bosh.io/docs/cli-envs.html#deployment-state), stored in `state.json`

		!!! note
			Subsequent runs of `deploy_bosh` will use `creds.yml` and `state.json` to apply changes to the BOSH environment.

After deploying the BOSH Director, continue to the [Deploying CFCR](../deploying-cfcr/) topic.