#Deploying BOSH for Kubo on vSphere

This topic describes how to deploy BOSH for Kubo on vSphere. Installing Kubo requires deploying a BOSH Director. 

After completing the procedures in this topic, continue to the [Configuring HAProxy for vSphere](routing-vsphere/) topic.

##Step 1: Create User Accounts

Log in to vCenter and then complete the procedures in the sections below to create the user accounts required for your Kubo installation.

###Create a BOSH User 

BOSH needs a user account with a particular set of privileges. This topic refers to this account as the "BOSH user." 

The role associated with the BOSH user must grant the following privileges:    
    
- **Data Store**
    - `Allocate space`
    - `Browse datastore`
    - `Low level file operations`
    - `Remove file`
    - `Update virtual machine files`
    - `Update virtual machine metadata`
- **Folder** 
- **Global**
    - `Manage custom attributes`
    - `Set custom attribute`
- **Host**
    - `Inventory`
        - `Modify cluster`
    - `Local operations`
- **Network**
- **Resource**
    - `Assign virtual machine to resource pool`
    - `Migrate powered off virtual machine`
    - `Migrate powered on virtual machine`
- **Virtual Machine**
    - `Configuration`
    - `Guest Operations`
    - `Interaction`
    - `Inventory`
    - `Provisioning`
    - `Service configuration`
    - `Snapshot management`
- **vApp**
- **vCenter Inventory Service**

For more information about user accounts and roles, see the [vSphere documentation](https://docs.vmware.com/en/VMware-vSphere/6.5/com.vmware.vsphere.security.doc/GUID-18071E9A-EED1-4968-8D51-E0B4F526FDA3.html).

###(Optional) Create a Persistence User

If you plan to provide your Kubernetes applications with access to persistent volumes, create a separate user account with a smaller set of privileges. This topic refers to this account as the "persistence user."

The role associated with the persistence user must grant the following privileges:    

- **Datastore**
    - `Allocate space`
    - `Low level file Operations`
- **Virtual Machine**
    - `Configuration`
        - `Add existing disk`
        - `Add or remove device`
        - `Remove disk`

If you plan to use the VSAN-policy-based volume provisioning feature in Kubernetes, the persistence user must grant the following additional privileges:
    
- **Network**
    - `Assign network`
- **Virtual machine**
    - `Configuration`
        - `Add new disk`
- **Virtual Machine**
    - `Inventory`
        - `Create new`
- **Resource**
    - `Assign virtual machine to resource pool`

##Step 2: Retrieve Information

Retrieve the following vSphere configuration details from vCenter. These values will be required later during the deployment of BOSH and Kubo.

- vCenter IP address
- Username and password for the BOSH user
- (optional) Username and password for the persistence user
- vSphere datacenter name
- Name of an existing cluster in the above datacenter
- Name of an existing datastore in the same datacenter
- Name of an existing resource pool in the cluster

##Step 3: Deploy BOSH

Perform the following steps to deploy BOSH for Kubo:

1. Ensure you are using a machine that has access to the VMs on the vSphere network. Depending on your network topology, you may need to execute the commands below on a bastion host.
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
   <p class="terminal">$ ./bin/generate_env_config "${kubo_env}" ${kubo_env_name} vsphere</p>
1. Navigate to `KUBO_ENV` and open the newly created `director.yml` file.
1. Populate the uncommented lines in the file to configure BOSH. You will need the vSphere configuration details you retrieved in [Step 2: Retrieve Information](#Step-2-Retrieve-Information).
1. In the same directory, open the `director-secrets.yml` file. 
1. Set the `vcenter_password` to the password for the BOSH user you created in [Step 1: Create User Accounts](#Create-a-BOSH-User).

	!!! warning
		The `director-secrets.yml` file contains sensitive information and should not be under version control.

1. Deploy the BOSH Director for Kubo. Enter the following command:

	<p class="terminal">$ ./bin/deploy_bosh "${kubo_env_path}"</p>
    
	The `deploy_bosh` script deploys a BOSH Director with all of the necessary components to install Kubo. 

	After the script completes, `KUBO_ENV` contains the following:

	* Credentials and SSL certificates for the BOSH Director, stored in `creds.yml`

		!!! warning
			The `cred.yml` file contains sensitive information and should not be under version control.

	* The [deployment state](https://bosh.io/docs/cli-envs.html#deployment-state), stored in `state.json`

		!!! note
			Subsequent runs of `deploy_bosh` will use `creds.yml` and `state.json` to apply changes to the BOSH environment.

After deploying the BOSH Director, continue to the [Configuring HAProxy for vSphere](routing-vsphere/) topic.