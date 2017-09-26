#Customizing Kubo

This topic describes how to deploy a customized Kubo installation. 

You can edit the [cloud config](https://bosh.io/docs/terminology.html#cloud-config) and [deployment manifest](https://bosh.io/docs/terminology.html#manifest) in order to modify your Kubo deployment.

Before completing the procedures in this topic, you must have deployed BOSH for Kubo and configured routing. For more information, see the [Preparing](/installing/#step-1-prepare-your-iaas) section for your IaaS.

!!! tip
	If you encounter problems when installing Kubo, see the [Troubleshooting Kubo](../managing/troubleshooting.md) topic.

##Step 1: Set Up Cloud Config

Perform the following steps to create and apply a modified cloud config:

1. Access your Kubo environment. This is the environment that contains the `KUBO_ENV` directory with your Kubo configuration. For more information, see the [Step 1: Access Your Kubo Environment](deploying-kubo/#step-1-access-your-kubo-environment).
1. In your Kubo environment, navigate to the `kubo-deployment` directory. Enter the following command:
	<p class="terminal">$ cd /share/kubo-deployment</p>
1. Run the following command:<br>
	`./bin/generate_cloud_config KUBO_ENV KUBO_ENV/cloud-config.yml`

	Where `KUBO_ENV` is the directory that contains the Kubo configuration.

	For example:
	<p class="terminal">$ ./bin/generate_cloud_config ~/kubo-env/kubo ~/kubo-env/kubo/cloud-config.yml</p>

1. Modify the generated cloud config file as necessary.
1. Log in to the BOSH Director as admin. Use the admin password located in `KUBO_ENV/creds.yml`. For example:
	<p class="terminal">$ bosh-cli -e my-kubo log-in
User (): admin
Password ():</p>
1. Update the BOSH Director with the customized cloud config. Run the following command:<br>
	`bosh-cli -e BOSH_NAME update-cloud-config KUBO_ENV/cloud-config.yml`

	Where:

	* `BOSH_NAME`: This is the name of your BOSH environment.
	* `KUBO_ENV`: This is the directory that contains the Kubo configuration.

	For example:
	<p class="terminal">$ bosh-cli -e my-bosh update-cloud-config ~/kubo-env/kubo/cloud-config.yml</p>

##Step 2: Generate Manifest

Perform the following steps to generate and modify the deployment manifest for Kubo:

1. In the `kubo-deployment` directory, run the following command:<br>
	`./bin/generate_kubo_manifest KUBO_ENV DEPLOYMENT_NAME  KUBO_ENV/kubo-manifest.yml`

	Where:

	* `KUBO_ENV`: This is the directory that contains the Kubo configuration.
	* `DEPLOYMENT_NAME`: Choose a name for your Kubo deployment.

	For example:
	<p class="terminal">$ ./bin/generate_kubo_manifest ~/kubo-env/kubo my-kubo > KUBO_ENV/kubo-manifest.yml</p>

1. Modify the generated Kubo manifest in the following ways:
	* Modify the manifest manually.
	* Substitute the variables in the manifest template by creating a variables file called `DEPLOYMENT_NAME-vars.yml` and placing it in the `KUBO_ENV` directory. In the variables file, specify the variables as key-value pairs. For example:
		```
		super-secret-secret: SuperSecretPa$$phrase
		```   
	* Manipulate parts of the manifest using [go-patch](https://github.com/cppforlife/go-patch/blob/master/docs/examples.md) [operations files](https://bosh.io/docs/cli-ops-files.html). Create an operations file file called `DEPLOYMENT_NAME.yml` and place it in the `KUBO_ENV` directory. Fill the file with go-patch instructions. For example:
		```
		- type: replace
		  path: /releases/name=etcd
		  value:
		    name: etcd
		    version: 0.99.0
    	```

1. If you created a variables file or an operations file, re-run the `./bin/generate_kubo_manifest` script, providing the same arguments as before. The script will automatically detect the variables file and operations file. For example:
	<p class="terminal">$ ./bin/generate_kubo_manifest ~/kubo-env/kubo my-kubo > KUBO_ENV/kubo-manifest.yml</p>

##Step 3: Deploy Kubo

To deploy Kubo, run the following command:<br>
`bosh-cli -e BOSH_NAME -d DEPLOYMENT_NAME deploy KUBO_ENV/kubo-manifest.yml`

Where:

* `BOSH_NAME`: This is the name of your BOSH environment.
* `DEPLOYMENT_NAME`: This is the name of your Kubo deployment.
* `KUBO_ENV`: This is the directory that contains the Kubo configuration.

For example:
<p class="terminal">$ bosh-cli -e my-bosh -d my-kubo deploy ~/kubo-env/kubo/kubo-manifest.yml</p>