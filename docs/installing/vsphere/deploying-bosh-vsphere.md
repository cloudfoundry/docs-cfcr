#Deploying BOSH for CFCR on vSphere

This topic describes how to deploy BOSH for Cloud Foundry Container Runtime (CFCR) on vSphere. Installing CFCR requires deploying a BOSH Director. 

After completing the procedures in this topic, continue to the [Deploying CFCR](../deploying-cfcr/) topic. 

!!! note
	CFCR was formerly known as Kubo, and many CFCR assets described in this topic still use the Kubo name.

##Step 1: Create User Accounts

Log in to vCenter and then complete the procedures in the sections below to create the user accounts required for your CFCR installation.

###Create a BOSH User 

BOSH needs a user account with a particular set of privileges. This topic refers to this account as the "BOSH user." 

The role associated with the BOSH user must grant the privileges described in the following table.

<table>
<tr>
<th>Privilege Type</th>    
<th>Privilege Name</th>
</tr>
<tr>
<td><strong>Datastore</strong></td>
<td><ul><li><code>Allocate space</code></li>
<li><code>Browse datastore</li></code>
<li><code>Low level file operations</li></code>
<li><code>Remove file</li></code>
<li><code>Update virtual machine files</li></code>
<li><code>Update virtual machine metadata</li></code>
</td>
</tr>
<tr>
<td><strong>Folder</strong></td>
<td>All</td>
</tr>
<tr>
<td><strong>Global</strong></td>
<td><ul><li><code>Manage custom attributes</code></li>
<li><code>Set custom attribute</code></li></ul>
</tr>
<tr>
<td><strong>Host</strong></td>
<td><ul><li><code>Inventory</code></li>
    <ul><li><code>Modify cluster</code></li></ul>
    <li><code>Local operations</code></li>
</ul>
</td>
</tr>
<tr>
<td><strong>Network</strong></td>
<td>All</td>
</tr>
<tr>
<td><strong>Resource</strong></td>
<td><ul><li><code>Assign virtual machine to resource pool</code></li>
    <li><code>Migrate powered off virtual machine</code></li>
    <li><code>Migrate powered on virtual machine</li></code>
</ul>
</td>
</tr>
<tr>
<td><strong>Virtual Machine</strong></td>
<td><ul><li><code>Configuration</code></li>
<li><code>Guest Operations</li></code>
<li><code>Interaction</li></code>
<li><code>Inventory</li></code>
<li><code>Provisioning</li></code>
<li><code>Service configuration</li></code>
<li><code>Snapshot management</li></code>
</ul>
</td>
</tr>
<tr>
<td><strong>vApp</strong></td>
<td>All</td>
</tr>
<tr>
<td><strong>vCenter Inventory Service</strong></td>
<td>All</td>
</tr>
</table>

For more information about user accounts and roles, see the [vSphere documentation](https://docs.vmware.com/en/VMware-vSphere/6.5/com.vmware.vsphere.security.doc/GUID-18071E9A-EED1-4968-8D51-E0B4F526FDA3.html).

###(Optional) Create a Persistence User

If you plan to provide your Kubernetes applications with access to persistent volumes, create a separate user account with a smaller set of privileges. This topic refers to this account as the "persistence user."

The role associated with the persistence user must grant the privileges described in the following table.    

<table>
<tr>
<th>Privilege Type</th>
<th>Privilege Name</th>
</tr>
<tr>
<td><strong>Datastore</strong></td>
<td><ul><li><code>Allocate space</code></li>
<li><code>Low level file Operations</code></li>
</ul>
</td>
</tr>
<tr>
<td><strong>Virtual Machine</strong></td>
<td><ul><li><code>Configuration</code></li>
<li><code>Add existing disk</code></li>
<li><code>Add or remove device</code></li>
<li><code>Remove disk</code></li>
</td>
</tr>
</table>

If you plan to use the VSAN-policy-based volume provisioning feature in Kubernetes, the persistence user must grant the additional privileges described in the following table.

<table>
<tr>
<th>Privilege Type</th>
<th>Privilege Name</th>
</tr>
<tr>
<td><strong>Network</strong></td>
<td><ul><li><code>Assign network</code></li></ul></td>
</tr>
<tr>
<td><strong>Virtual Machine</strong></td>
<td><ul><li><code>Configuration</code></li>
    <ul><li><code>Add new disk</code></li></ul>
</li></ul>
<ul><li><code>Inventory</code></li>
    <ul><li><code>Create new</code></li></ul>
</td>
</tr>
<tr>
<td><strong>Resource</strong></td>
<td><ul><li><code>Assign virtual machine to resource pool</code></li></ul></td>
</tr>
</table>

##Step 2: Retrieve Information

Retrieve the following vSphere configuration details from vCenter. These values will be required later during the deployment of BOSH and CFCR.

- vCenter IP address
- Username and password for the BOSH user
- (optional) Username and password for the persistence user
- vSphere datacenter name
- Name of an existing cluster in the above datacenter
- Name of an existing datastore in the same datacenter
- Name of an existing resource pool in the cluster

##Step 3: Generate a Configuration Template

Perform the following steps to generate a CFCR configuration template:

1. Ensure you are using a machine that has access to the VMs on the vSphere network. Depending on your network topology, you may need to execute the commands below on a bastion host.
1. Change into the home directory. Enter the following command:
	<p class="terminal">$ cd ~</p>
1. Get the latest version of `kubo-deployment`. Enter the following command:
	<p class="terminal">$ wget http<span>s:/</span>/storage.googleapis.com/kubo-public/kubo-deployment-latest.tgz</p>
1. Expand the tarball. Enter the following command:
	<p class="terminal">$ tar -xvf kubo-deployment-latest.tgz</p>
1. Change into `kubo-deployment`. Enter the following command:
	<p class="terminal">$ cd ~/kubo-deployment</p>
1. Set three environment variables with the following commands:
	<p class="terminal">$ export kubo_env=~/kubo-env
$ export kubo_env_name=kubo
$ export kubo_env_path="\${kubo_env}/\${kubo_env_name}"</p>

    !!! note
		`kubo_env_path` points to the directory containing the CFCR configuration. Later topics will refer to this path as `KUBO_ENV`.

1. Make a new directory path with the following command:
	<p class="terminal">$ mkdir -p "${kubo_env}"</p>
1. Generate a CFCR configuration template:
   <p class="terminal">$ ./bin/generate_env_config "${kubo_env}" ${kubo_env_name} vsphere</p>

##Step 4: Configure Routing

If you want to configure Cloud Foundry to handle routing for CFCR, perform the procedures in [Configuring Cloud Foundry Routing](../cf-routing/).

If you want to configure HAProxy to handle routing for CFCR, perform the procedures in [Configuring HAProxy](../haproxy/).

!!! note 
	vSphere does not have first-party load-balancing support. You can configure HAProxy to handle external access to the Kubernetes master nodes for administration traffic, and the Kubernetes worker nodes for application traffic.

If you want to configure an external load balancer, perform the following steps:

1. Navigate to `KUBO_ENV` and open the newly created `director.yml` file.
1. Under the `IaaS routing mode settings` section, set the `routing_mode` to `external`.
1. Set the `kubernetes_master_host` to the IP address of your external load balancer.
1. Set the `kubernetes_master_port` to the port exposed by the external load balancer.
1. Comment out the `master_target_pool` property.

##Step 5: Deploy BOSH

Perform the following steps to deploy a BOSH Director:

1. Continue working in the `director.yml` file to configure BOSH by populating the remaining uncommented lines. You will need the vSphere configuration details you retrieved in [Step 2: Retrieve Information](#Step-2-Retrieve-Information).
1. In the same directory, open the `director-secrets.yml` file. 
1. Set the `vcenter_password` to the password for the BOSH user you created in [Step 1: Create User Accounts](#Create-a-BOSH-User).

	!!! warning
		The `director-secrets.yml` file contains sensitive information and should not be under version control.

1. Ensure your vCenter hostname can be resolved using `8.8.8.8` as the nameserver. For example:

	<p class="terminal">$ dig @8.8.8.8 ab2-host-a101-11.foo-12.bar.cf-example.com</p>

	If the hostname cannot be resolved, open `~/kubo-deployment/bosh-deployment/bosh.yml` and locate the `dns` property. Replace `8.8.8.8` with the nameserver of your vSphere environment.

1. Deploy the BOSH Director for CFCR. Enter the following command:

	<p class="terminal">$ ./bin/deploy_bosh "${kubo_env_path}"</p>
    
	The `deploy_bosh` script deploys a BOSH Director with all of the necessary components to install CFCR. 

	After the script completes, `KUBO_ENV` contains the following:

	* Credentials and SSL certificates for the BOSH Director, stored in `creds.yml`

		!!! warning
			The `creds.yml` file contains sensitive information and should not be under version control.

	* The [deployment state](https://bosh.io/docs/cli-envs.html#deployment-state), stored in `state.json`

		!!! note
			Subsequent runs of `deploy_bosh` will use `creds.yml` and `state.json` to apply changes to the BOSH environment.

After deploying the BOSH Director, continue to the [Deploying CFCR](../deploying-cfcr/) topic.