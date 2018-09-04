#What is CFCR?

This topic provides an overview of Cloud Foundry Container Runtime (CFCR).

CFCR is a [BOSH release](https://github.com/cloudfoundry-incubator/kubo-release) that offers a uniform way to instantiate, deploy, and manage highly available Kubernetes clusters on a cloud platform using BOSH. 

You can deploy CFCR on Google Cloud Platform (GCP), vSphere, Amazon Web Services (AWS), and OpenStack.

CFCR relies on **Kubernetes** and **BOSH**:

* **Kubernetes** is an open-source platform designed to automate deploying, scaling, and operating application containers. For more information, see the [Kubernetes documentation](https://kubernetes.io/docs/home/).
* **BOSH** is an open-source tool for release engineering, deployment, lifecycle management, and monitoring of distributed systems. For more information, see the [BOSH documentation](https://bosh.io/docs).

!!! note
	CFCR consumes the standard open-source distribution of Kubernetes, not a custom distribution. 

!!! note
	CFCR was formerly known as Kubo, and many CFCR assets still use the Kubo name. CFCR does not require Cloud Foundry.

##What problems does CFCR solve?

CFCR aims to solve a range of problems that operators and developers face when deploying and managing fleets of Kubernetes clusters.

###Deploying Solutions

CFCR offers the following solutions to improve the experience of deploying Kubernetes clusters:

* **Repeatability and consistency** when deploying “Kubernetes-as-a-service” within an organization
* **Plans, quotas, and other constraints** defined by service operators
* **A single control plane** to provision and manage Kubernetes services

###Managing Solutions

CFCR offers the following solutions to improve the experience of managing Kubernetes clusters:

* **High-availability and multi-AZ support**: BOSH can deploy multiple master/etcd/worker nodes across multiple availability zones, and monitor their health.
* **Scaling**: BOSH allows the operator to scale the number of instances in the cluster up and down by modifying the manifest.
* **VM Healing**: BOSH continuously monitors the health of all VM instances and recreates VMs. 
Self-healing VMs and monitoring via BOSH.
* **Upgrades**: BOSH manages the rolling upgrade process for a fleet of Kubernetes clusters.
