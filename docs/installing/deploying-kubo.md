#Deploying Kubo

This topic describes how to deploy Kubo.

Before completing the procedures in this topic, you must have deployed BOSH for Kubo and configured routing. For more information, see the [Preparing](/installing/#step-1-prepare-your-iaas) section for your IaaS.

!!! tip
	If you encounter problems when installing Kubo, see the [Troubleshooting Kubo](../managing/troubleshooting.md) topic.

##Step 1: Access Your Kubo Environment

Access your Kubo environment. This is the environment that contains the `KUBO_ENV` directory with your Kubo configuration.

Follow the steps for your IaaS:

* **Google Cloud Platform (GCP)**: Open the Google Cloud Shell and use `gcloud compute ssh` to SSH into your bastion VM. For more information, see the [(Optional) Step 1: Set Up Your Environment](./gcp/routing-gcp/#optional-step-1-set-up-your-environment) section of the <em>Configuring IaaS Routing for GCP</em>.
* **Amazon Web Services (AWS)**: SSH onto the bastion VM from your local machine by performing the following the steps:
	1. Change into your Terraform working directory with the following command:
		<p class="terminal">$ cd ~/kubo-deployment/docs/user-guide/platforms/aws</p>
	1. SSH onto the bastion VM with the following command:
		<p class="terminal">$ ssh -i ~/deployer.pem ubuntu@$(terraform output bosh-bastion-ip)</p>
* **vSphere or OpenStack**: If you deployed BOSH for Kubo from a bastion VM, SSH into the VM. Otherwise, navigate to the Kubo environment on your local machine.

##(Optional) Step 2: Configure Proxy Access

If you want to configure Docker to allow proxy access, edit the `KUBO_ENV/director.yml` file to set the `http_proxy`, `https_proxy`, and `no_proxy` variables. 

The `http_proxy` is the proxy server for HTTP traffic, `https_proxy` is the proxy server for HTTPS traffic, and `no_proxy` specifies addresses to exclude from proxying.

For example:

```
http_proxy: http://my.proxy.local:73636
https_proxy: https://secure.proxy.local:5566
no_proxy: '192.0.2.0,192.0.2.1'
```

##Step 3: Deploy Kubo

Perform the following steps to deploy Kubo:

1. In your Kubo environment, navigate to the `kubo-deployment` directory. Enter the following command:
	<p class="terminal">$ cd /share/kubo-deployment</pre> 
1. Execute the `deploy_k8s` script to download the packages necessary to deploy a Kubernetes cluster, and to spin up the master, worker, and etcd nodes. 

	Run the following command:<br>
	`./bin/deploy_k8s KUBO_ENV CLUSTER_NAME [RELEASE_SOURCE]`<br><br>
	Where:<br>

	* `KUBO_ENV`: This is the directory that contains the Kubo configuration.
	* `CLUSTER_NAME`: Choose a unique name for your Kubernetes cluster.
	* `RELEASE_SOURCE`. Optionally select one of the following values to specify where to find the Kubo BOSH release. If you do not select an option, the script defaults to `local`: 
		* `dev`: Manually builds a development release from the local version of the `kubo-deployment` repo
		* `public`: Uses the published precompiled release on the internet, downloaded from the URL supplied by the `kubo_release_url` variable in `KUBO_ENV/director.yml`
		* `local`: Uses the local tarball release
		* `skip`: Uses the release already uploaded to the BOSH Director

	For example:
	<p class="terminal">$ ./bin/deploy_k8s ~/kubo-env/kubo my-cluster</pre>

##Step 4: Access Kubernetes

After deploying the cluster, perform the following steps: 

1. Execute the `set_kubeconfig` script to configure `kubectl`, the Kubernetes command line interface. Specify your `KUBO_ENV` and the name of your new cluster. For example: 

	<p class="terminal">$ ./bin/set_kubeconfig ~/kubo-env/kubo my-cluster</p>

1. Verify that the settings have been applied correctly by listing the Kubernetes pods in the `kubo-system` namespace. Enter the following command:

	<p class="terminal">$ kubectl get pods --namespace=kube-system</p>

	If you have successfully configured `kubectl`, the output resembles the following:
	<p class="terminal">NAME                                    READY     STATUS    RESTARTS   AGE
	heapster-2736291043-9rw42               1/1       Running   0          2d
	kube-dns-3329716278-dpdj0               3/3       Running   0          2d
	kubernetes-dashboard-1367211859-jq9mw   1/1       Running   0          2d
	monitoring-influxdb-564852376-67fdd      1/1       Running   0          2d
	</p>

For more information about using `kubectl`, see the [Kubernetes documentation](https://kubernetes.io/docs/user-guide/kubectl-overview/).

##Step 5: Enable Application Access

The procedure for exposing applications run by the Kubernetes cluster to the internet varies depending on your routing configuration.

###IaaS Routing

If you configured IaaS load balancers to handle routing for Kubo, you can expose routes using the service type `LoadBalancer` for your Kubernetes deployments. See the [Kubernetes documentation](https://kubernetes.io/docs/tutorials/kubernetes-basics/expose-intro/) for more information.

!!! warning 
	Any resources that are provisioned by Kubernetes will not be deleted by BOSH if you delete your Kubo deployment. 

###Cloud Foundry Routing

If you configured Cloud Foundry to handle routing for Kubo, perform the following steps to expose both TCP and HTTP routes to the Kubernetes services:

!!! note
	You must expose your Kubernetes services using a single `NodePort`. See the [Kubernetes documentation](https://kubernetes.io/docs/tutorials/kubernetes-basics/expose-intro/) for more information.
1. Add a label to your service named `tcp-route-sync`, and set the value of the label to the front-end port you want to expose your application on. The front-end port must be within the range you configured when enabling TCP routing in Cloud Foundry. See the [Enabling TCP Routing](http://docs.cloudfoundry.org/adminguide/enabling-tcp-routing.html) topic in the Cloud Foundry documentation for more information.
<br><br>For example:
	<p class="terminal">$ kubectl label services my-service tcp-route-sync=443</p>
	
	!!! note
		It may take up to 60 seconds to create the route.

1. Use a browser to access your service at `CLOUD-FOUNDRY-TCP-URL`:`FRONT-END-PORT`.
1. Add a label to your service named `http-route-sync`, and set the value of the label to the route you want to create for your application. For example:
	<p class="terminal">$ kubectl label services my-service http-route-sync=web-app</p>
	
	!!! note
		It may take up to 60 seconds to create the route.

1. Use a browser to access your service at `ROUTE-NAME.CLOUD-FOUNDRY-APPS-DOMAIN`.

##(Optional) Step 6: Configure Persistence

Kubo clusters support the following Kubernetes volume types:

* `emptyDir`
* `hostPath`
* `gcePersistentDisk`
* `VsphereVolume`
* `AWSElasticBlockStore`

For more information about volumes, see the [Kubernetes documentation](https://kubernetes.io/docs/concepts/storage/volumes/).

To use storage in the Kubo clusters, you must configure the `cloud-provider` job on the master and worker nodes. See the `cloud-provider` [spec](https://github.com/cloudfoundry-incubator/kubo-release/blob/master/jobs/cloud-provider/spec) in `kubo-release` for more information on the properties that are required for each `cloud-provider` type.

For more information on configuring Kubernetes to access storage for your `cloud-provider` type, see the [Kubo documentation](https://kubernetes.io/docs/concepts/storage/persistent-volumes/).

!!! warning 
	Any resources that are provisioned by Kubernetes will not be deleted by BOSH if you delete your Kubo deployment. 


