#

![CFCR Logo](./images/common/cfcr-full.png)

This is the documentation for Cloud Foundry Container Runtime (CFCR), an open-source project that provides a solution for deploying and managing [Kubernetes](https://kubernetes.io/docs/home/) clusters using [BOSH](https://bosh.io/docs).

CFCR uses BOSH to improve the Kubernetes experience for both operators and developers. By leveraging BOSH, CFCR adds capabilities such as high availability, scaling, and self-healing to Kubernetes clusters.

CFCR was formerly known as Kubo, and many CFCR assets still use the Kubo name. CFCR does not require Cloud Foundry, but you can optionally configure a Cloud Foundry deployment to handle routing for CFCR.

See [What is CFCR?](./overview/what-is-cfcr/) to learn more.

## Installing and Configuring

**DEPRECATED**
>We no longer support the following strategies for deploying BOSH and CFCR.
To deploy BOSH, see [BOSH Bootloader](https://github.com/cloudfoundry/bosh-bootloader).
To deploy CFCR, see the [CFCR repository](https://github.com/cloudfoundry-incubator/kubo-release) on GitHub.

## Managing and Troubleshooting

The following topics describe how to manage and troubleshoot CFCR.

[Deleting CFCR](./managing/deleting/)

## Certifications

<img alt="Kubernetes 1.11 Certification logo" src="https://github.com/cncf/artwork/raw/master/kubernetes/certified-kubernetes/1.11/color/certified-kubernetes-1.11-color.png" width="198px">

Kubernetes® is a registered trademark of The Linux Foundation in the United States and other countries, and is used pursuant to a license from The Linux Foundation.
