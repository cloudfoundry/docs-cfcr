# Cloud Foundry Container Runtime

This topic describes Cloud Foundry Container Runtime (CFCR), an open-source project that provides a solution for deploying and managing [Kubernetes](https://kubernetes.io/docs/home/) clusters using [BOSH](https://bosh.io/docs).

## What is CFCR?

CFCR is a BOSH release that offers a uniform way to instantiate, deploy, and manage highly available Kubernetes clusters on a cloud platform using BOSH.

CFCR relies on **Kubernetes** and **BOSH**:

* **Kubernetes** is an open-source platform designed to automate deploying, scaling, and operating application containers. For more information, see the [Kubernetes documentation](https://kubernetes.io/docs/home/).
* **BOSH** is an open-source tool for release engineering, deployment, lifecycle management, and monitoring of distributed systems. For more information, see the [BOSH documentation](https://bosh.io/docs).

CFCR uses BOSH to improve the Kubernetes experience for both operators and developers. By leveraging BOSH, CFCR adds capabilities such as high availability, scaling, and self-healing to Kubernetes clusters.

!!! note
	CFCR was formerly known as Kubo, and many CFCR assets still use the Kubo name. CFCR does not require Cloud Foundry but you can optionally configure a Cloud Foundry deployment to handle routing for CFCR.

CFCR consumes the standard open-source distribution of Kubernetes, not a custom distribution.

You can deploy CFCR on Google Cloud Platform (GCP), vSphere, Amazon Web Services (AWS), and OpenStack.

## What problems does CFCR solve?

CFCR aims to solve a range of problems that operators and developers face when deploying and managing fleets of Kubernetes clusters.

### Deploying Solutions

CFCR offers the following solutions to improve the experience of deploying Kubernetes clusters:

* **Repeatability and consistency** when deploying “Kubernetes-as-a-service” within an organization
* **A single control plane** to provision and manage Kubernetes services

### Managing Solutions

CFCR offers the following solutions to improve the experience of managing Kubernetes clusters:

* **High-availability and multi-AZ support**: BOSH can deploy multiple master/etcd/worker nodes across multiple availability zones, and monitor their health.
* **Scaling**: BOSH allows the operator to scale the number of instances in the cluster up and down by modifying the manifest.
* **VM healing**: BOSH continuously monitors the health of all VM instances and recreates VMs.
Self-healing VMs and monitoring via BOSH.
* **Upgrades**: BOSH manages the rolling upgrade process for a fleet of Kubernetes clusters.

## How do I install and configure CFCR?

To install CFCR, you must first deploy and configure BOSH.
You can deploy BOSH using the BOSH Bootloader (`bbl`) command-line utility.
See the [BOSH Bootloader repository](https://github.com/cloudfoundry/bosh-bootloader) for instructions.

After configuring BOSH, you can deploy the CFCR BOSH release.
See the [CFCR repository](https://github.com/cloudfoundry-incubator/kubo-release) for instructions.

## Certifications

<img alt="Kubernetes 1.11 Certification logo" src="https://github.com/cncf/artwork/raw/master/kubernetes/certified-kubernetes/1.13/color/certified-kubernetes-1.13-color.png" width="198px">

Kubernetes® is a registered trademark of The Linux Foundation in the United States and other countries, and is used pursuant to a license from The Linux Foundation.
