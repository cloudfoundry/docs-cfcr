#Deploying CFCR

This topic describes how to deploy Cloud Foundry Container Runtime (CFCR).

Before completing the procedures in this topic, you must have deployed BOSH for CFCR and configured routing. For more information, see the [Preparing](/installing/#step-1-prepare-your-iaas) section for your IaaS.

This topic describes how to deploy a standard CFCR installation. If you want to deploy a customized CFCR installation, see the [Customizing CFCR](customizing-cfcr/) topic.

!!! note
	CFCR was formerly known as Kubo, and many CFCR assets described in this topic still use the Kubo name.

!!! tip
	If you encounter problems when installing CFCR, see the [Troubleshooting CFCR](../managing/troubleshooting.md) topic.

##Step 1: Access Your CFCR Environment

Access your CFCR environment. This is the environment that contains the `KUBO_ENV` directory with your CFCR configuration.

Follow the steps for your IaaS:

* **Google Cloud Platform (GCP)**: Open the Google Cloud Shell and use `gcloud compute ssh` to SSH into your bastion VM. For more information, see the [(Optional) Step 1: Set Up Your Environment](./gcp/routing-gcp/#optional-step-1-set-up-your-environment) section of the <em>Configuring IaaS Routing for GCP</em>.
* **Amazon Web Services (AWS)**: SSH onto the bastion VM from your local machine by performing the following the steps:
	1. Change into your Terraform working directory with the following command:
		<p class="terminal">$ cd ~/kubo-deployment/docs/user-guide/platforms/aws</p>
	1. SSH onto the bastion VM with the following command:
		<p class="terminal">$ ssh -i ~/deployer.pem ubuntu@$(terraform output bosh-bastion-ip)</p>
* **vSphere or OpenStack**: If you deployed BOSH for CFCR from a bastion VM, SSH into the VM. Otherwise, navigate to the CFCR environment on your local machine.

##(Optional) Step 2: Configure Proxy Access

If you want to configure Docker to allow proxy access, edit the `KUBO_ENV/director.yml` file to set the `http_proxy`, `https_proxy`, and `no_proxy` variables.

!!! note
	You must perform this procedure if you have proxy access requirements to reach internal or external Docker registries. 

The `http_proxy` is the proxy server for HTTP traffic, `https_proxy` is the proxy server for HTTPS traffic, and `no_proxy` specifies addresses to exclude from proxying.

For example:

```
http_proxy: http://my.proxy.local:73636
https_proxy: https://secure.proxy.local:5566
no_proxy: '192.0.2.0,192.0.2.1'
```

CFCR passes these settings to the Docker Engine daemon that runs in the CFCR workers.

##Step 3: Deploy CFCR

Perform the following steps to deploy CFCR:

1. In your CFCR environment, navigate to the `kubo-deployment` directory. Enter the following command:
	<p class="terminal">$ cd /share/kubo-deployment</pre> 
1. Execute the `deploy_k8s` script to download the packages necessary to deploy a Kubernetes cluster, and to spin up the master, worker, and etcd nodes. 

	Run the following command:<br>
	`./bin/deploy_k8s KUBO_ENV CLUSTER_NAME [RELEASE_SOURCE]`<br><br>
	Where:<br>

	* `KUBO_ENV`: This is the directory that contains the CFCR configuration.
	* `CLUSTER_NAME`: Choose a unique name for your Kubernetes cluster.
	* `RELEASE_SOURCE`. Optionally select one of the following values to specify where to find the BOSH release. If you do not select an option, the script defaults to `local`: 
		* `dev`: Manually builds a development release from the local version of the `kubo-release` repo
		* `public`: Uses the published precompiled release on the internet, downloaded from the URL supplied by the `kubo_release_url` variable in `KUBO_ENV/director.yml`
		* `local`: Uses the local tarball release
		* `skip`: Uses the release already uploaded to the BOSH Director

	For example:
	<p class="terminal">$ ./bin/deploy_k8s ~/kubo-env/kubo my-cluster</p>
	
1. If you configured an external load balancer for [vSphere](vsphere/deploying-bosh-vsphere/#step-4-configure-routing) or [OpenStack](openstack/deploying-bosh-openstack/#step-2-configure-routing), perform the following steps:

	1. [Download](https://bosh.io/) the latest version of the BOSH CLI v2 to your CFCR environment and place the binary in your PATH.
	1. Create an alias for your BOSH Director, by running the following command:<br>

		`bosh alias-env YOUR_ENV -e BOSH_DIRECTOR_IP` 

		Where:

		* `YOUR_ENV`: Choose an alias.
		* `BOSH_DIRECTOR_IP`: Enter the IP address for the BOSH Director for CFCR.

		For example:
		<p class="terminal">$ bosh alias-env my-cfcr -e 10.0.0.3</p>

	1. List your deployments and retrieve the name of your CFCR deployment, which may be named `kubo`. For example:
		<p class="terminal">$ bosh -e my-cfcr deployments</p>
	1. List the VMs in your deployment, specifying the name of your deployment retrieved above. For example:
		<p class="terminal">$ bosh -e my-cfcr -d kubo vms</p>
	1. From the above output, retrieve the IP addresses for the master node VMs. Add these IP addresses to your external load balancer.

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

##Step 5: Enable Application Access

The procedure for exposing applications run by the Kubernetes cluster to the internet varies depending on your routing configuration.

###IaaS Routing

If you configured IaaS load balancers to handle routing for CFCR, you can expose routes using the service type `LoadBalancer` for your Kubernetes deployments. See the [Kubernetes documentation](https://kubernetes.io/docs/tutorials/kubernetes-basics/expose-intro/) for more information.

!!! warning 
	Any resources that are provisioned by Kubernetes will not be deleted by BOSH if you delete your CFCR deployment. 

###Cloud Foundry Routing

If you [configured Cloud Foundry](cf-routing/) to handle routing for CFCR, perform the following steps to expose both TCP and HTTP routes to the Kubernetes services:

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

Persistent storage in CFCR requires configuring the `cloud-provider` job on the master and worker nodes. On AWS, GCP, and vSphere, CFCR deployments have the `cloud-provider` job enabled by default. For more information, see the `cloud-provider` [spec](https://github.com/cloudfoundry-incubator/kubo-release/blob/master/jobs/cloud-provider/spec) in `kubo-release` and the [Kubernetes documentation](https://kubernetes.io/docs/concepts/storage/persistent-volumes/).

### Create a StorageClass

CFCR supports [dynamic volume provisioning](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#dynamic), to allow storage volumes to be created on-demand. The dynamic provisioning feature eliminates the need to pre-provision persistent storage.

To enable this feature for your cluster, you must define one or more [StorageClass](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#storageclasses) objects. Each StorageClass object enables operators to define and expose a different class of storage in the cluster.

Perform the following steps to define a StorageClass:

1. Download the StorageClass spec for your IaaS to your CFCR environment. Run the command for your IaaS:

	* **AWS**: `wget https://raw.githubusercontent.com/pivotal-cf-experimental/kubo-ci/master/specs/storage-class-aws.yml`
	* **GCP**: `wget https://raw.githubusercontent.com/pivotal-cf-experimental/kubo-ci/master/specs/storage-class-gcp.yml`
	* **vSphere**: `wget https://raw.githubusercontent.com/pivotal-cf-experimental/kubo-ci/master/specs/storage-class-vsphere.yml`

1. Apply the StorageClass. Run `kubectl create -f STORAGE-CLASS-SPEC.yml` where `STORAGE-CLASS-SPEC` is the name of your StorageClass spec file. For example:
	<p class="terminal">$ kubectl create -f storage-class-gcp.yml</p>

You can also enable more than one type of persistent volume with additional StorageClasses. See the [Kubernetes documentation](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#storageclasses) for more information.

### Create a PersistentVolumeClaim

A [PersistentVolumeClaim](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims) is a request for storage by a user. A claim enables a user to define the desired [access mode](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes) and storage capacity, among other configuration details. 

Typically a user will configure different claims for different workloads, depending on the storage requirements of an application.

The following procedure applies an example PersistentVolumeClaim that uses the above StorageClass. After applying the PersistentVolumeClaim, Kubernetes creates the requested volume dynamically when the user deploys a pod.

Perform the following steps:

1. Download the example PersistentVolumeClaim spec. Run the following command:
	<p class="terminal">$ wget http<span>s:</span>//raw.githubusercontent.com/pivotal-cf-experimental/kubo-ci/master/specs/persistent-volume-claim.yml</p>
1. Modify the storage size of the volume to meet the requirements of your workload by editing the value for `spec.resources.storage`. See the following example:
	```
	spec:
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 2Gi
      storageClassName: ci-storage
	```

1. Apply the PersistentVolumeClaim. Run the following command:
	<p class="terminal">$ kubectl create -f persistent-volume-claim.yml</p>
1. Confirm that the PersistentVolumeClaim is configured. Run the following command:
	<p class="terminal">$ kubectl get pvc -o wide</p>
	The output should resemble the following:
	<p class="terminal">NAME       STATUS    VOLUME                                     CAPACITY   ACCESSMODES   STORAGECLASS   AGE
ci-claim   Bound     pvc-3e11131e-a1a0-11a1-3a2a-0a111e11111e   1Gi        RWO           ci-storage     </p>
1. Create a pod with persistent storage by providing a configuration file that specifies the newly created PersistentVolumeClaim. See the `pv-guestbook.yml` configuration [file](https://github.com/pivotal-cf-experimental/kubo-ci/blob/master/specs/pv-guestbook.yml) as an example. For more information about configuring a pod for persistent storage, see the [Kubernetes documentation](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/).

	Run `kubectl create -f POD-CONFIGURATION.yml` where `POD-CONFIGURATION` is the name of the pod configuration file. For example: 
	<p class="terminal">$ kubectl create -f pv-guestbook.yml</p>

!!! tip
	You can also create persistent volumes by pre-provisioning storage at the IaaS level and later declaring a persistent volume manually.

### Additional Information

CFCR clusters support the following Kubernetes volume types:

* `emptyDir`
* `hostPath`
* `gcePersistentDisk`
* `VsphereVolume`
* `AWSElasticBlockStore`

For more information about volumes, see the [Kubernetes documentation](https://kubernetes.io/docs/concepts/storage/volumes/).

!!! warning 
	Any resources that are provisioned by Kubernetes will not be deleted by BOSH if you delete your CFCR deployment. 


