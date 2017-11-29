#Configuring Cloud Foundry Routing

This topic describes how to configure Cloud Foundry to handle routing for Cloud Foundry Container Runtime (CFCR). 

You configure Cloud Foundry routing by editing the BOSH configuration files before deploying BOSH for CFCR. The procedure for generating these files and using them to deploy BOSH for CFCR will vary depending on your IaaS.

##Prerequisites

Consult the following list of prerequisites before performing the procedures in this topic:

* You must have a running Cloud Foundry deployment. For more information, see the [Deploying Cloud Foundry](https://docs.cloudfoundry.org/deploying/index.html) section of the Cloud Foundry documentation.

* You must have completed both the pre-deployment and post-deployment procedures in the [Enabling TCP Routing](http://docs.cloudfoundry.org/adminguide/enabling-tcp-routing.html) topic of the Cloud Foundry documentation to enable TCP routing in your Cloud Foundry deployment. Ensure that you have created a shared domain for the TCP domain and configured a quota for TCP routes.

* You must have installed the [UAA Command Line Client](https://github.com/cloudfoundry/cf-uaac) (UAAC). 

* You must have completed the following procedures specific to your IaaS:

	* If you are deploying CFCR on GCP, you must have completed [Step 1: Set Up Your Shell Environment](./gcp/deploying-bosh-gcp/#step-1-set-up-your-shell-environment) through [Step 4: Generate BOSH Configuration](./gcp/deploying-bosh-gcp/#step-4-generate-bosh-configuration) of the *Deploying BOSH for CFCR on GCP* topic.
	* If you are deploying CFCR on vSphere, you must have completed [Step 1: Create User Accounts](./vsphere/deploying-bosh-vsphere/#step-1-create-user-accounts) through [Step 3: Generate a Configuration Template](./vsphere/deploying-bosh-vsphere/#step-3-generate-a-configuration-template) of the *Deploying BOSH for CFCR on vSphere* topic. 
	* If you are deploying CFCR on AWS, you must have completed [Step 1: Set Up Your Shell Environment](./aws/deploying-bosh-aws/#step-1-set-up-your-shell-environment) through [Step 4: Create IAM User](./aws/deploying-bosh-aws/#step-4-create-iam-user) of the *Deploying BOSH for CFCR on AWS* topic.
	* If you are deploying CFCR on OpenStack, you must have completed [Step 1: Generate a Configuration Template](./openstack/deploying-bosh-openstack/#step-1-generate-a-configuration-template) of the *Deploying BOSH for CFCR on OpenStack* topic.

##Step 1: Enable Internal Communication

You must edit your IaaS firewall rules to enable communication between the Cloud Foundry components and the CFCR VMs. The procedures will vary by IaaS, but you must do the following:

* Ensure that the Cloud Foundry routing components can reach your CFCR cluster on the `NodePort` port range.
* Ensure that the Cloud Foundry TCP Router can reach the CFCR master nodes on port 8443 to communicate with the Kubernetes API server.
* Ensure that the CFCR components can reach the Cloud Foundry NATS servers on port 4222.
* Ensure that the CFCR components can reach the Cloud Foundry API and UAA endpoints. Both are HTTPS endpoints in the Cloud Foundry system domain, accessible on port 443. 

##Step 2: Create a Routing UAA Client

Perform the following steps to create a UAA client for CFCR routing:

1. Target the UAA server of your Cloud Foundry deployment. Run the following command:<br>
	`uaac target uaa.YOUR-SYS-DOMAIN --skip-ssl-validation`

	Where `YOUR-SYS-DOMAIN` is your system domain.

	For example:
	<p class="terminal">$ uaac target uaa.sys.example.com --skip-ssl-validation</p>

1. Authenticate and obtain an access token for the admin client. Enter the following command:
	<p class="terminal">$ uaac token client get admin</p>
	When prompted, enter the UAA admin client password. This is **uaa:admin:client_secret** in your Cloud Foundry deployment manifest.

1. Add a client for CFCR routing. CFCR will use this client to create routes in Cloud Foundry. Enter the following command: 
	<p class="terminal">$ uaac client add routing_api_client \
--authorities "routing.router_groups.read,routing.routes.write,cloud_controller.admin" --authorized_grant_type "client_credentials"</p>
	When prompted, enter a secret for the new client. Record this secret.

##Step 3: Configure CFCR for Cloud Foundry Routing

Perform the following steps to configure CFCR for Cloud Foundry routing:

1. Navigate to `KUBO_ENV` and open the `director-secrets.yml` file.
1. Uncomment the `routing-cf-client-secret` line and fill in the UAA routing client secret you created above.
1. Uncomment the `routing-cf-nats-password` line and fill in the NATS password. This is **nats: nats: password** in your Cloud Foundry deployment manifest.

	!!! warning
		The `director-secrets.yml` file contains sensitive information and should not be under version control.

1. Open the `director.yml` file.
1. Comment out the `IaaS routing mode settings` section.
1. Uncomment the `CF routing mode settings` section, and set appropriate values for your deployment.
    1. Uncomment `routing_mode: cf`. 
    1. Set the `kubernetes_master_host` to the TCP router hostname or IP address for Cloud Foundry. This is typically `tcp.YOUR-APPS-DOMAIN`, such as `tcp.apps.cf-example.com`.
	
	    !!! tip
		    If you are using a domain, ensure that the DNS resolves correctly. For more information, see the [Pre-Deployment Steps](https://docs.cloudfoundry.org/adminguide/enabling-tcp-routing.html#-pre-deployment-steps) section of the <em>Enabling TCP Routing</em> topic in the Cloud Foundry documentation.

    1. Set the `kubernetes_master_port` to an available port on the Cloud Foundry TCP router.
    1. Set the `routing-cf-api-url` to the Cloud Foundry API URL, such as `https://api.sys.cf-example.com`.
    1. Set the `routing-cf-client-id` to `routing_api_client`.
    1. Set the `routing-cf-uaa-url` to the Cloud Foundry UAA URL, such as `https://uaa.sys.cf-example.com`.
    1. Set the `routing-cf-app-domain-name` to the Cloud Foundry apps domain, such as `apps.cf-example.com.`
    1. Set the `routing-cf-nats-internal-ips` to the array of internal IP addresses used by Cloud Foundry NATS, such as `[192.168.16.13]`. To obtain the IP addresses for your NATS instances, log in to the BOSH Director you used to deploy Cloud Foundry and run `bosh -e YOUR-ENV instances`.
    1. Uncomment `routing-cf-nats-username: nats`.
    1. Uncomment `routing-cf-nats-port: 4222`.

1. Deploy BOSH for CFCR by performing the procedures specific to your IaaS:
	* [Deploy on GCP](./gcp/deploying-bosh-gcp/#step-5-deploy-bosh-director)
	* [Deploy on vSphere](./vsphere/deploying-bosh-vsphere/#step-5-deploy-bosh)
	* [Deploy on AWS](./aws/deploying-bosh-aws/#step-5-deploy-bosh-director)
	* [Deploy on OpenStack](./openstack/deploying-bosh-openstack/#step-3-deploy-bosh)
