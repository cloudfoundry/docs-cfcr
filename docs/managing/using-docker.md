#Using Docker Registries

This topic describes how to use Docker registries with Kubo.

##Use Private Docker Registries

To use a private Docker registry with Kubo, you must add the registry CA certificate to the BOSH Director manifest and redeploy the BOSH Director. Then the BOSH Director will store the certificate in every Kubernetes node it provisions.

1. Access your Kubo environment. This is the environment that contains the `KUBO_ENV` directory with your Kubo configuration. For more information, see the [Step 1: Access Your Kubo Environment](https://docs-kubo.cfapps.io/installing/deploying-kubo/#step-1-access-your-kubo-environment) section of the <em>Deploying Kubo</em> topic.
1. Open the `director.yml` file. This is the BOSH Director manifest.
1. Under `properties.director.trusted_certs,` add the registry CA certificate.
	```
	properties:
	  director:
	    trusted_certs: |
	    # Private Docker registry CA certificate
	    -----BEGIN CERTIFICATE-----
        MIICsjCCAhugAwIBAgIJAMcyGWdRwnFlMA0GCSqGSIb3DQEBBQUAMEUxCzAJBgNV
        BAYTAkFVMRMwEQYDVQQIEwpTb21lLVN0YXRlMSEwHwYDVQQKExhJbnRlcm5ldCBX
        ...
        ItuuqKphqhSb6PEcFMzuVpTbN09ko54cHYIIULrSj3lEkoY9KJ1ONzxKjeGMHrOP
        KS+vQr1+OCpxozj1qdBzvHgCS0DrtA==
        -----END CERTIFICATE-----
    ```

	For more information about configuring trusted certificates in BOSH, see the [BOSH documentation](https://bosh.io/docs/trusted-certs.html).

1. Redeploy the BOSH Director by following the procedures specific to your IaaS: 
	* [Deploying BOSH for Kubo on GCP](https://docs-kubo.cfapps.io/installing/gcp/deploying-bosh-gcp/#step-4-deploy-bosh-director)
	* [Deploying BOSH for Kubo on vSphere](https://docs-kubo.cfapps.io/installing/vsphere/deploying-bosh-vsphere/#step-4-deploy-bosh)
	* [Deploying BOSH for Kubo on AWS](https://docs-kubo.cfapps.io/installing/aws/deploying-bosh-aws/#step-5-deploy-bosh-director)
	* [Deploying BOSH for Kubo on OpenStack](https://docs-kubo.cfapps.io/installing/openstack/deploying-bosh-openstack/#step-4-deploy-bosh)