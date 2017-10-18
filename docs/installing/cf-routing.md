#Configuring Cloud Foundry Routing

This topic describes how to configure Cloud Foundry to handle routing for Kubo. 

You configure Cloud Foundry routing by editing the BOSH configuration files before deploying BOSH for Kubo. The procedure for generating these files and using them to deploy BOSH for Kubo will vary depending on your IaaS.

After finishing this topic, continue to the [Deploying Kubo](deploying-kubo/) topic.

##Prerequisites

Consult the following list of prerequisites before performing the procedures in this topic:

* You must have a running Cloud Foundry deployment. For more information, see the [Deploying Cloud Foundry](https://docs.cloudfoundry.org/deploying/index.html) section of the Cloud Foundry documentation.

* You must have deployed Kubo to the same network as Cloud Foundry.

* You must have completed both the pre-deployment and post-deployment procedures in the [Enabling TCP Routing](http://docs.cloudfoundry.org/adminguide/enabling-tcp-routing.html) topic of the Cloud Foundry documentation to enable TCP routing in your Cloud Foundry deployment. Ensure that you have created a shared domain for the TCP domain and configured a quota for TCP routes.

* You must have installed the [UAA Command Line Client](https://github.com/cloudfoundry/cf-uaac) (UAAC). 

* You must have [prepared your IaaS for Kubo](/installing/#step-1-prepare-your-iaas) to the point of generating a `director.yml` and `director-secrets.yml` file in your `KUBO_ENV`, but not yet deploying a BOSH Director.

##Step 1: Enable Internal Communication

You must edit your IaaS firewall rules to enable communication between the Cloud Foundry components and the Kubo VMs. The procedures will vary by IaaS, but you must do the following:

* Ensure that the Cloud Foundry routing components can reach your Kubo cluster on the `NodePort` port range.
* Ensure that the Cloud Foundry TCP Router can reach the Kubo master nodes on port 8443 to communicate with the Kubernetes API server.
* Ensure that the Kubo components can reach the Cloud Foundry NAT servers on port 4222.
* Ensure that the Kubo components can reach the Cloud Foundry API and UAA endpoints. Both are HTTPS endpoints in the Cloud Foundry system domain, accessible on port 443. 

##Step 2: Create a Routing UAA Client

Perform the following steps to create a UAA client for Kubo routing:

1. Target the UAA server of your Cloud Foundry deployment. Run the following command:<br>
	`uaac target uaa.YOUR-SYS-DOMAIN --skip-ssl-validation`

	Where `YOUR-SYS-DOMAIN` is your system domain.

	For example:
	<p class="terminal">$ uaac target uaa.sys.example.com --skip-ssl-validation</p>

1. Authenticate and obtain an access token for the admin client. Enter the following command:
	<p class="terminal">$ uaac token client get admin</p>
	When prompted, enter the UAA admin client password. This is **uaa:admin:client_secret** in your Cloud Foundry deployment manifest.

1. Add a client for Kubo routing. Kubo will use this client to create routes in Cloud Foundry. Enter the following command: 
	<p class="terminal">$ uaac client add routing_api_client \
--authorities "routing.router_groups.read,routing.routes.write,cloud_controller.admin" --authorized_grant_type "client_credentials"</p>
	When prompted, enter a secret for the new client. Record this secret.

##Step 3: Configure Kubo for Cloud Foundry Routing

Perform the following steps to configure Kubo for Cloud Foundry routing:

1. Navigate to `KUBO_ENV` and open the `director-secrets.yml` file.
1. Uncomment the `routing-cf-client-secret` line and fill in the UAA routing client secret you created above.
1. Uncomment the `routing-cf-nats-password` line and fill in the NATS password. This is **nats: nats: password** in your Cloud Foundry deployment manifest.

	!!! warning
		The `director-secrets.yml` file contains sensitive information and should not be under version control.

1. Open the `director.yml` file.

1. Uncomment the `CF routing mode settings` section and comment out the `IaaS routing mode settings`.
1. Set the `kubernetes_master_host` to the TCP router hostname or IP address for Cloud Foundry. This is typically `tcp.YOUR-APPS-DOMAIN`, such as `tcp.apps.cf-example.com`.

	!!! tip
		If you are using a domain, ensure that the DNS resolves correctly. For more information, see the [Pre-Deployment Steps](https://docs.cloudfoundry.org/adminguide/enabling-tcp-routing.html#-pre-deployment-steps) section of the <em>Enabling TCP Routing</em> topic in the Cloud Foundry documentation.

1. Leave the `kubernetes_master_port` commmented out. This property is ignored when deploying Kubo with Cloud Foundry routing.
1. Set the `routing-cf-api-url` to the Cloud Foundry API URL, such as `https://api.sys.cf-example.com`.
1. Set the `routing-cf-client-id` to `routing_api_client`.
`. 
1. Set the `routing-cf-uaa-url` to the Cloud Foundry UAA URL, such as `https://uaa.sys.cf-example.com`.
1. Set the `routing-cf-app-domain-name` to the Cloud Foundry apps domain, such as `apps.cf-example.com.`
1. Set the `routing-cf-nats-internal-ips` to the array of internal IP addresses used by Cloud Foundry NATS, such as `[192.168.16.13]`. To obtain the IP addresses for your NATS instances, log in to the BOSH Director you used to deploy Cloud Foundry and run `bosh -e YOUR-ENV instances`.

After configuring the `director-secrets.yml` and `director.yml` files, deploy BOSH for Kubo with the `deploy_bosh` script by following the [procedures for your IaaS](/installing/#step-1-prepare-your-iaas). 
 


