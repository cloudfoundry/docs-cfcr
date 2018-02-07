#Deploying BOSH for CFCR on AWS

This topic describes how to deploy BOSH for Cloud Foundry Container Runtime (CFCR) on Amazon Web Services (AWS). Installing CFCR requires deploying a BOSH Director. 

In the procedures below, you use [Terraform](https://www.terraform.io/docs/) to pave your infrastructure and create a bastion VM. Then you deploy a BOSH Director from the bastion VM. 

!!! note
	CFCR was formerly known as Kubo, and many CFCR assets described in this topic still use the Kubo name.

##Step 1: Set Up Your Shell Environment

1. Install the latest version of [Terraform](https://www.terraform.io/intro/getting-started/install.html). You must have Terraform v0.10.2 or greater to perform the procedures in this topic.
1. Log in to the AWS Console and create an EC2 key pair named `deployer`. For more information, see the [AWS documentation](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html#having-ec2-create-your-key-pair).
1. The private key file is automatically downloaded by your browser. Move the file to your home directory by entering the following command:
	<p class="terminal">$ mv ~/Downloads/deployer.pem ~/deployer.pem</p>
1. Change the permissions of the private key file. Enter the following command:
	<p class="terminal">$ chmod 600 ~/deployer.pem</p>
1. If you are deploying CFCR more than once, you must set a unique prefix for every installation. Export an environment variable to set a prefix for your CFCR installation. Use letters and dashes only. For example:
	<p class="terminal">$ export prefix=my-kubo</p>
1. Set the access key ID and secret access key for an IAM user with the AdministratorAccess policy as environment variables named `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`. For example:
	<p class="terminal">$ export AWS_ACCESS_KEY_ID=AKIDZAAFAA0AOABCJ00Z
$ export AWS_SECRET_ACCESS_KEY=dsfSDFKOSDFKOasdmasdKSADOK</p>
1. Set the ID of your VPC as an environment variable named `vpc_id`. This VPC must be in the zone where you want to deploy CFCR, with a CIDR range that has at least an `/22` netmask. For example:
	<p class="terminal">$ export vpc_id=vpc-2111120e</p>

	!!! tip
		To retrieve the ID for a VPC, navigate to the VPC Dashboard of the AWS Console and locate the **VPC ID** column.

	!!! tip
		If you plan to use [Cloud Foundry to handle routing](../cf-routing.html) for CFCR, deploy CFCR in the same VPC as Cloud Foundry.

1. Set the name of the private key to use on CFCR VMs and the location of the private key file as environment variables named `key_name` and `private_key_filename`. Enter the following commands:
	<p class="terminal">$ export key_name=deployer
$ export private_key_filename="\${HOME}/\${key_name}.pem"</p>

1. Set the region and zone where you want to deploy CFCR as environment variables named `region` and `zone`. For example:
	<p class="terminal">$ export region=us-west-2
$ export zone=us-west-2a</p>

1. Set the IP address prefix of the public subnet that will be used by the bastion VM, NAT gateway, and load balancers as an environment variable named `public_subnet_ip_prefix`. This prefix must be within the VPC CIDR range. For example:
	<p class="terminal">$ export public_subnet_ip_prefix="10.0.0"</p>

1. Set the IP address prefix of the private subnet that will be used for CFCR VMs and the BOSH Director as an environment variable named `private_subnet_ip_prefix`. This prefix must be within the VPC CIDR range. For example:
	<p class="terminal">$ export private_subnet_ip_prefix="10.0.1"</p>

	!!! note
		You create the public and private subnets with Terraform in a later step, so you do not need to create them in the AWS Console.

1. Set the path of the Terraform state file as an environment variable named `kubo_terraform_state`. Enter the following command:
	<p class="terminal">$ export kubo_terraform_state="${HOME}/terraform.tfstate"</p>

##Step 2: Deploy Bastion VM

Perform the following steps to deploy a bastion VM with a set of security group rules that secures access to the CFCR deployment:

1. Change into the home directory. Enter the following command:
	<p class="terminal">$ cd ~</p>
1. See the [release notes](../../overview/release-notes) for a link to the latest `kubo-deployment` release. Enter the following command, replacing `KUBO-RELEASE-URL` with the release artifact URL:
	<p class="terminal">$ wget http<span>s:/</span>/KUBO-RELEASE-URL.tgz</p>
1. Expand the tarball. Enter the following command, replacing `KUBO-RELEASE` with the name of the file you downloaded in the previous step:
	<p class="terminal">$ tar -xvf KUBO-RELEASE.tgz</p>
1. Change into the directory that contains the AWS Terraform templates. Enter the following command:
	<p class="terminal">$ cd kubo-deployment/docs/terraform/aws/platform</p>
1. From the AWS Console, check if your VPC has an existing Internet Gateway (IGW) attached. If it does, perform the following steps to edit the Terraform template:
	1. Open `kubo-infrastructure.tf`.
	1. Use `//` to comment out the `aws_internet_gateway` section so that it looks like the following:
		```
		//resource "aws_internet_gateway" "gateway" {
		//    vpc_id = "${var.vpc_id}"
		//}
		```
	1. Retrieve the value of your VPC's IGW from the AWS Console.
	1. Under `resource "aws_route_table" "public"`, set the value of `gateway_id` to the value of your VPC's IGW. For example:

		```
		resource "aws_route_table" "public" {
		...
		    route {
		      cidr_block = "0.0.0.0/0"
		      gateway_id = "igw-871b9de1"
		    }
		}
		```
		
1. Initialize the Terraform working directory. Enter the following command:
	<p class="terminal">$ terraform init</p>
1. Create the bastion and other resources. Enter the following command:
<p class="terminal">$ terraform apply \
  -var region="\${region}" \
  -var zone="\${zone}" \
  -var vpc_id="\${vpc_id}" \
  -var prefix="\${prefix:-cfcr}" \
  -var public_subnet_ip_prefix="\${public_subnet_ip_prefix}" \
  -var private_subnet_ip_prefix="\${private_subnet_ip_prefix}" \
  -var private_key_filename="\${private_key_filename}" \
  -var key_name="\${key_name}" \
  -state=\${kubo_terraform_state}
</p>

	This command takes between 60 and 90 seconds to complete.

	!!! tip
		To preview the Terraform execution plan before applying it, run `plan` instead of `apply`.

	!!! warning
		Subsequent runs of `terraform apply` will delete the bastion VM.

##Step 3: Generate CFCR Configuration

1. SSH onto the bastion VM. Enter the following command:
	<p class="terminal">$ ssh -i ~/deployer.pem ubuntu@$(terraform output bosh-bastion-ip)</p>
1. Change into the root of the `kubo-deployment` repo. Enter the following command:
	<p class="terminal">cd /share/kubo-deployment</p> 
1. Set three Kubo environment variables with the following commands:
	<p class="terminal">$ export kubo_envs=~/kubo-env
$ export kubo_env_name=kubo
$ export kubo_env_path="\${kubo_envs}/\${kubo_env_name}"</p>

	!!! note
		`kubo_env_path` points to the directory containing the CFCR configuration. Later topics will refer to this path as `KUBO_ENV`.

1. Make a new directory path with the following command:
	<p class="terminal">$ mkdir -p "${kubo_envs}"</p>

1. Generate a CFCR configuration template. Enter the following command:
	<p class="terminal">$ ./bin/generate_env_config "\${kubo_envs}" "\${kubo_env_name}" aws</p>
1. Apply the default network settings configured during the infrastructure paving to the template. Enter the following commands:
	<p class="terminal">$ /usr/bin/update_aws_env "${kubo_env_path}/director.yml"</p>

	!!! tip
		You can directly edit the configuration file located at `${kubo_env_path}/director.yml`.

##Step 4: Create IAM User

Perform the following steps to create a new IAM user to deploy the BOSH Director:

1. Log in to the AWS Console and navigate to the IAM Management Console.
1. In the left navigation, click **Users**.
1. Click **Add user**.
1. Enter a user name, such as `kubo-bosh`, and select **Programmatic access** for **Access type**. 
1. Click **Next: Permissions**.
1. Skip the permissions section by clicking **Next: Review**.
1. Click **Create user**.
1. Locate the **Access key ID** column and record the access key ID.
1. Locate the **Secret access key** column and click **Show**. Record the secret access key.
1. Click **Close**.
1. Click the newly created user from the list.
1. Locate the value for **User ARN** and record the string of digits. For example, if the value is `arn:aws:iam::711111112123:user/kubo-bosh`, record `711111112123`.
1. Click **Add inline policy**.
1. Select **Custom Policy** and click **Select**.
1. Enter a **Policy Name**, such as `kubo-bosh-policy`.
1. Copy the following policy and paste it under **Policy Document**, replacing `YOUR-ACCOUNT-ID` with the string of digits you recorded from the value for **User ARN**:

	```
	{
	    "Version": "2012-10-17",
	    "Statement": [
	        {
	            "Action": [
	                "ec2:AssociateAddress",
	                "ec2:AttachVolume",
	                "ec2:CreateVolume",
	                "ec2:DeleteSnapshot",
	                "ec2:DeleteVolume",
	                "ec2:DescribeAddresses",
	                "ec2:DescribeImages",
	                "ec2:DescribeInstances",
	                "ec2:DescribeRegions",
	                "ec2:DescribeSecurityGroups",
	                "ec2:DescribeSnapshots",
	                "ec2:DescribeSubnets",
	                "ec2:DescribeVolumes",
	                "ec2:DetachVolume",
	                "ec2:CreateSnapshot",
	                "ec2:CreateTags",
	                "ec2:RunInstances",
	                "ec2:TerminateInstances",
	                "ec2:RegisterImage",
	                "ec2:DeregisterImage",
	                "elasticloadbalancing:*"
	            ],
	            "Effect": "Allow",
	            "Resource": "*"
	        },
	        {
	            "Effect": "Allow",
	            "Action": "iam:PassRole",
	            "Resource": "arn:aws:iam::YOUR-ACCOUNT-ID:role/*cfcr*"
	        }
	    ]
	}
	```

1. Click **Apply Policy**.

	!!! warning
		If you want to configure Cloud Foundry to handle routing for CFCR, **do not continue to the next section**. Perform the procedures in [Configuring Cloud Foundry Routing](../cf-routing/) before deploying the BOSH Director. 

##Step 5: Deploy BOSH Director

Perform the following steps to deploy a BOSH Director from the bastion VM:

1. Open `${kubo_env_path}/director-secrets.yml` and fill in the `access_key_id` and `secret_access_key` for your newly created user.

	!!! warning
		The `director-secrets.yml` file contains sensitive information and should not be under version control.

1. Deploy the BOSH Director for CFCR. Enter the following command:
	<p class="terminal">$ /share/kubo-deployment/bin/deploy_bosh "${kubo_env_path}" ~/deployer.pem</p>

	After the script completes, `KUBO_ENV` contains the following:

	* Credentials and SSL certificates for the BOSH Director, stored in `creds.yml`

		!!! warning
			The `creds.yml` file contains sensitive information and should not be under version control.

	* The [deployment state](https://bosh.io/docs/cli-envs.html#deployment-state), stored in `state.json`

		!!! note
			Subsequent runs of `deploy_bosh` will use `creds.yml` and `state.json` to apply changes to the BOSH environment.

If you plan to use IaaS routing for CFCR, continue to [Configure IaaS Routing for AWS](routing-aws/).

If you have configured Cloud Foundry routing, continue to [Deploying CFCR](../deploying-cfcr/).
