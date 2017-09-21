#Deploying BOSH for Kubo on OpenStack

This topic describes how to deploy BOSH for Kubo on OpenStack. Installing Kubo requires deploying a BOSH Director. 

After completing the procedures in this topic, continue to the [Deploying Kubo](../deploying-kubo/) topic. 

##Step 1: Generate a Configuration Template

Perform the following steps to generate a Kubo configuration template:

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
		`kubo_env_path` points to the directory containing the Kubo configuration. Later topics will refer to this path as `KUBO_ENV`.

1. Make a new directory path with the following command:
	<p class="terminal">$ mkdir -p "${kubo_env}"</p>
1. Generate a Kubo configuration template:
   <p class="terminal">$ ./bin/generate_env_config "${kubo_env}" ${kubo_env_name} openstack</p>

##Step 2: Configure HAProxy

OpenStack does not have first-party load-balancing support. But you can configure HAProxy to handle external access to the Kubernetes master nodes for administration traffic, and the Kubernetes worker nodes for application traffic.

Perform the following steps to configure HAProxy:

1. Navigate to `KUBO_ENV` and open the newly created `director.yml` file.
1. Uncomment the `Proxy routing mode settings` section and comment the `IaaS routing mode settings`.
1. Set the `kubernetes_master_host` to the IP address of the HAProxy master node. This must be a floating IP, allocated to the project and not associated with any other instances.
1. Set the `kubernetes_master_port` to the port for the Kubernetes API server on the HAProxy master node.
1. Set the `worker_haproxy_ip_addresses` to the IP address(es) of the HAProxy worker node(s). This must be a floating IP, allocated to the project and not associated with any other instances.
1. Set the `worker_haproxy_tcp_frontend_port` to the front-end port for the HAProxy TCP pass-through.
1. Set the `worker_haproxy_tcp_backend_port` to the back-end port for the HAProxy TCP pass-through.

The current implementation of HAProxy routing is a single-port TCP pass-through. In order to route traffic to multiple Kubernetes services, use an Ingress controller. For more information, see the [Kubernetes documentation](https://kubernetes.io/docs/concepts/services-networking/ingress/) and the [Ingress examples readme](https://github.com/kubernetes/ingress/tree/master/examples#ingress-examples)  in the Kubernetes GitHub repo.

##Step 4: Deploy BOSH

Perform the following steps to deploy a BOSH Director:

1. Continue working in the `director.yml` file to configure BOSH by populating the remaining uncommented lines. For `default_key_name`, enter the name of an OpenStack key pair, and ensure you have a copy of the PEM-encoded private key file on your machine.
1. In the same directory, open the `director-secrets.yml` file.
1. Set the `openstack_password` to the OpenStack admin user password.

	!!! warning
		The `director-secrets.yml` file contains sensitive information and should not be under version control.

1. Deploy the BOSH Director for Kubo, supplying the path to the private key file as an argument. This is the private key whose name you set as `default_key_name` in `director.yml`. For example:

	<p class="terminal">$ ./bin/deploy_bosh "${kubo_env_path}" private_key.pem</p>

	The `deploy_bosh` script deploys a BOSH Director with all of the necessary components to install Kubo. 

	After the script completes, `KUBO_ENV` contains the following:

	* Credentials and SSL certificates for the BOSH Director, stored in `creds.yml`

		!!! warning
			The `cred.yml` file contains sensitive information and should not be under version control.

	* The [deployment state](https://bosh.io/docs/cli-envs.html#deployment-state), stored in `state.json`

		!!! note
			Subsequent runs of `deploy_bosh` will use `creds.yml` and `state.json` to apply changes to the BOSH environment.

After deploying the BOSH Director, continue to the [Deploying Kubo](../deploying-kubo/) topic.