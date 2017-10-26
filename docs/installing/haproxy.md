#Configuring HAProxy

This topic describes how to configure HAProxy to handle routing for Cloud Foundry Container Runtime (CFCR) on vSphere and OpenStack. 

vSphere and OpenStack do not have first-party load-balancing support. But you can configure HAProxy to handle external access to the Kubernetes master nodes for administration traffic, and the Kubernetes worker nodes for application traffic.

The current implementation of HAProxy routing is a single-port TCP pass-through. In order to route traffic to multiple Kubernetes services, use an Ingress controller. For more information, see the [Kubernetes documentation](https://kubernetes.io/docs/concepts/services-networking/ingress/) and the [Ingress examples readme](https://github.com/kubernetes/ingress/tree/master/examples#ingress-examples)  in the Kubernetes GitHub repo.

##Configure HAProxy for vSphere

Before configuring HAProxy for vSphere, you must have completed the procedures in [Step 1: Create User Accounts](./vsphere/deploying-bosh-vsphere/#step-1-create-user-accounts) to [Step 3: Generate a Configuration Template](./vsphere/deploying-bosh-vsphere/#step-3-generate-a-configuration-template) of the *Deploying BOSH for CFCR on vSphere* topic.

Perform the following steps to configure HAProxy for vSphere:

1. Navigate to `KUBO_ENV` and open the newly created `director.yml` file.
1. Uncomment the `Proxy routing mode settings` section and comment out the `IaaS routing mode settings` section.
1. Set the `kubernetes_master_host` to the IP address of the HAProxy master node.
1. Set the `kubernetes_master_port` to the port for the Kubernetes API server on the HAProxy master node.
1. Set the `worker_haproxy_ip_addresses` to the IP address(es) of the HAProxy worker node(s).
1. Set the `worker_haproxy_tcp_frontend_port` to the front-end port for the HAProxy TCP pass-through.
1. Set the `worker_haproxy_tcp_backend_port` to the back-end port for the HAProxy TCP pass-through.
1. Perform the procedures in [Step 5: Deploy BOSH](./vsphere/deploying-bosh-vsphere/#step-5-deploy-bosh) of the *Deploying BOSH for CFCR on vSphere* topic to deploy a BOSH Director.

##Configure HAProxy for OpenStack

Before configuring HAProxy for OpenStack, you must have completed the procedures in [Step 1: Generate a Configuration Template](./openstack/deploying-bosh-openstack/#step-1-generate-a-configuration-template) of the *Deploying BOSH for CFCR on OpenStack* topic.

Perform the following steps to configure HAProxy for OpenStack:

1. Navigate to `KUBO_ENV` and open the newly created `director.yml` file.
1. Uncomment the `Proxy routing mode settings` section and comment the `IaaS routing mode settings`.
1. Set the `kubernetes_master_host` to the IP address of the HAProxy master node. This must be a floating IP, allocated to the project and not associated with any other instances.
1. Set the `kubernetes_master_port` to the port for the Kubernetes API server on the HAProxy master node.
1. Set the `worker_haproxy_ip_addresses` to the IP address(es) of the HAProxy worker node(s). This must be a floating IP, allocated to the project and not associated with any other instances.
1. Set the `worker_haproxy_tcp_frontend_port` to the front-end port for the HAProxy TCP pass-through.
1. Set the `worker_haproxy_tcp_backend_port` to the back-end port for the HAProxy TCP pass-through.
1. Perform the procedures in [Step 3: Deploy BOSH](./openstack/deploying-bosh-openstack/#step-3-deploy-bosh) of the *Deploying BOSH for CFCR on OpenStack* topic to deploy a BOSH Director.
