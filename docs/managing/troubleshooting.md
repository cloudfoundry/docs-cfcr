#Troubleshooting CFCR

This topic describes how to troubleshoot commonly encountered problems with [installing](#troubleshoot-a-failed-installation) and [running](#troubleshoot-a-running-installation) Cloud Foundry Container Runtime (CFCR).

!!! note "Note"
    CFCR was formerly known as Kubo, and many CFCR assets described in this topic still use the Kubo name.

##Troubleshoot a Failed Installation

### Unable to Create Google Storage Bucket When Deploying BOSH

#### Symptom

When [deploying BOSH for CFCR on Google Cloud Platform (GCP)](../installing/gcp/deploying-bosh-gcp/), you cannot create a Google Storage Bucket. An error similar to the following occurs:

<p class="terminal">CmdError{"type":"Bosh::Clouds::CloudError","message":
"Creating stemcell: Creating Google Storage Bucket: 
Post https://www.googleapis.com/storage/v1/b?alt=json\u0026project=kubo: 
Get http://192.0.2.0/computeMetadata/v1/instance/service-accounts/default/token: 
dial 192.0.2.0:80: i/o timeout","ok_to_retry":false}
</p>

#### Explanation

Your service account for Terraform may be misconfigured.

#### Solution

Ensure you have created a service account for Terraform, and that you have successfully configured it, by performing the procedures in the [Step 1: Set Up Your Shell Environment](../installing/gcp/deploying-bosh-gcp/#step-1-set-up-your-shell-environment) and [Step 2: Set Up a GCP Account for Terraform](../installing/gcp/deploying-bosh-gcp/#step-2-set-up-a-gcp-account-for-terraform) sections of the <em>Deploying BOSH for CFCR on GCP</em> topic.

---

### Service Account Permission Errors When Deploying BOSH

#### Symptom

When [deploying BOSH for CFCR on GCP](../installing/gcp/deploying-bosh-gcp/), you encounter service account permission issues.

#### Explanation

Your service account for Terraform may not have the correct permissions.

#### Solution

Ensure you have granted the Terraform service account owner access to your project, by performing the procedures in the [Step 2: Set Up a GCP Account for Terraform](../installing/gcp/deploying-bosh-gcp/#step-2-set-up-a-gcp-account-for-terraform) section of the <em>Deploying BOSH for CFCR on GCP</em> topic.

You can also verify that the correct permissions are applied by logging into the GCP Console and navigating to the **IAM & admin** section. 

If the permissions are set correctly but you are still experiencing permission issues, try creating a new service account for Terraform with a different name.

---

### I/O Timeout When Deploying BOSH

#### Symptom

When deploying BOSH through a `sshuttle` connection, an error similar to the following occurs:

<p class="terminal">Command 'deploy' failed:
  Deploying:
    Creating instance 'bosh/0':
      Waiting until instance is ready:
        Sending ping to the agent:
          Performing request to agent endpoint 'https://mbus:294a691d057ede1af4f696aab36c4bc5@10.0.0.4:6868/agent':
            Performing POST request:
              Post https://mbus:294a691d057ede1af4f696aab36c4bc5@10.0.0.4:6868/agent: dial tcp 10.0.0.4:6868: i/o timeout
</p>

In the above example, `10.0.0.4` is the IP address of the BOSH Director.

#### Explanation

The TCP connection has timed out.

#### Solution

Restart the `sshuttle` connection.

---

### Unable to Log In to CredHub

#### Symptom

Deploying BOSH or CFCR fails because CredHub authentication fails. The following error occurs:

<p class="terminal">The provided username and password combination are incorrect. Please validate your input and retry your request.</p>

CredHub is a credential management system deployed with BOSH. For more information, see the CredHub GitHub [repo](https://github.com/cloudfoundry-incubator/credhub).

#### Explanation

This is typically caused by a version mismatch between the CredHub CLI and the CredHub server. The versions must match exactly.

#### Solution

Ensure that the version of the CredHub CLI installed on the machine you deployed BOSH from matches the version of the CredHub server. Enter the following command:

<p class="terminal">$ credhub --version
CLI Version: 0.4.0
Server Version: 0.4.0</p>

If the versions do not match, update the CredHub CLI on the machine you deployed BOSH from. See the CredHub CLI GitHub [repo](https://github.com/cloudfoundry-incubator/credhub-cli) for more information.

---

### Master Not Running After Update

#### Symptom

When running the `deploy_k8s` script to deploy a Kubernetes cluster, you encounter an error similar to the following:

<p class="terminal">Updating instance master: master/20f5c31f-4329-46a7-ae03-484f0a17f6a3 (0) (canary) (00:01:14)
            Error: 'master/0 (20f5c31f-4329-46a7-ae03-484f0a17f6a3)' is not running after update. 
            Review logs for failed jobs: kubernetes-api, kubernetes-controller-manager, kubernetes-scheduler, kubernetes-api-route-registrar
Error: 'master/0 (20f5c31f-4329-46a7-ae03-484f0a17f6a3)' is not running after update. 
            Review logs for failed jobs: kubernetes-api, kubernetes-controller-manager, kubernetes-scheduler, kubernetes-api-route-registrar
</p>

#### Explanation

This error occurs if any of the jobs deployed in the master VM fail to start. This typically happens when using Cloud Foundry to handle routing for CFCR. 

#### Solution

Try the following solutions:

* Check the fields `routing-cf-client-id` in `KUBO_ENV/director.yml` and `routing-cf-client-secret` in `KUBO_ENV/director-secrets.yml` to ensure that the UAAC credentials that you are using are valid. You can use the [UAAC CLI](https://docs.cloudfoundry.org/adminguide/uaa-user-management.html) to create and manage credentials.
* Check  that the route to the TCP routing API URL is accessible. The URL is defined in the field `routing-cf-api-url` in `KUBO_ENV/director.yml`.

### Kubernetes Deployment Fails After BOSH is Redeployed

#### Symptom

When deploying a Kubernetes cluster with the `deploy_k8s` script following a redeployment of BOSH, you encounter an error similar to the following:

<p class="terminal">$ bin/deploy_k8s $KUBO_ENV my-kubo public
=====================================
|     BOSH K8S Cluster Deployer     |
=====================================

Fetching info:
  Performing request GET 'https://10.0.7.4:25555/info':
    Performing GET request:
      Retry: Get https://10.0.7.4:25555/info: x509: certificate signed by unknown authority (possibly because of "crypto/rsa: verification error" while trying to verify candidate authority certificate "ca")

Exit code 1</p>

#### Explanation

The SSL certificate is signed by unknown authority.

#### Solution

Reset your BOSH alias to use a new SSL certificate. Run the `set_bosh_alias` script from the same directory where you ran `deploy_k8s`. For example:

<p class="terminal">$ bin/set_bosh_alias $KUBO_ENV</p>

---

### Worker Failure When Deploying Second Cluster

#### Symptom

A worker fails when deploying a second Kubernetes cluster. You encounter an error similar to the following:

<p class="terminal">Updating instance worker: worker/aca2bddf-59b1-4802-a9c5-e09aa09a0efd (0) (canary) (00:03:57)
L Error: Action Failed get_task: Task 75bce522-4fba-4295-7b8c-d63d86f7dcc6 result: 1 of 1 post-start scripts failed. Failed Jobs: kubelet.
</p>

#### Explanation

The master port for the second Kubernetes cluster may be in use.

#### Solution

Ensure the port specified as `kubernetes-master-port` is not already in use by another Kubernetes cluster. You can override the value in `director.yml` using variables files for different clusters.   

---

### Deprecated Disk Property

#### Symptom

When using Terraform to [provision the bastion VM and other resources](https://docs-kubo.cfapps.io/installing/gcp/deploying-bosh-gcp/) required to deploy CFCR on GCP, you encounter an error similar to the following:

<p class="terminal">2 error(s) occurred:

* google_compute_instance.bosh-bastion: "disk": [REMOVED] Use boot_disk, scratch_disk, and attached_disk instead
* google_compute_instance.nat-instance-private-with-nat-primary: "disk": [REMOVED] Use boot_disk, scratch_disk, and attached_disk instead</p>

#### Explanation

This error is due to a deprecated `disk` property in Terraform.

#### Solution

From the Google Cloud Shell, download the latest version of `kubo-deployment`. Enter the following command:

<p class="terminal">$ wget https://storage.googleapis.com/kubo-public/kubo-deployment-latest.tgz
</p> 

You can also update your existing `kubo-infrastructre.tf` file. Replace the `disk` block with the following:

```
boot_disk {
  initialize_params = {
    image = "${var.latest_ubuntu}"
  }
}
```

### Other Connectivity Issues

#### Symptom

Various connectivity issues may arise during deployment.

#### Explanation

Your host names may not be resolving correctly.

#### Solution

Ensure that all the host names used in the configuration are resolving to the correct IP addresses.

---

##Troubleshoot a Running Installation

### UAA 401 Error When Using the BOSH CLI v2

#### Symptom

When using the BOSH CLI v2, you encounter a 401 error from the User Account and Authentication (UAA) server. An error similar to the following occurs:

<p class="terminal">$ bosh-cli -e my-kubo deployments
Using environment 'my-kubo.example.com' as '?'

Finding deployments:
  Performing request GET 'https://my-kubo.example.com:25555/deployments':
    Performing GET request:
      Refreshing token: UAA responded with non-successful status code '401' response '{"error":"invalid_token","error_description":"The token expired, was revoked, or the token ID is incorrect: acc3bc0eeb2b4323995f6d5873d9f52e-r"}'

Exit code 1
</p>

BOSH for CFCR deploys a UAA server to handle user management. For more information, see the [BOSH documentation](https://bosh.io/docs/director-users-uaa.html).

#### Explanation

You may not be logged in to the BOSH Director.

#### Solution

Ensure you are logged in to the BOSH Director as admin. For example:
<p class="terminal">$ bosh-cli -e my-bosh log-in
User (): admin
Password ():
</p>

Use the admin password located in `KUBO_ENV/creds.yml`.

