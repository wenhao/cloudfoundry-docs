<!--
##Bootstrapping an AWS Environment for Cloud Foundry
-->

<!--
This topic describes how to set up an environment for Cloud Foundry on Amazon Web Services (AWS) using `bosh aws create`. This command-line bootstrap tool creates all of the structures that underlie your deployment.
-->

<!--
The `bosh aws create` command creates many structures within AWS. If it does not complete successfully, or you want to abort the process, you can clean up the AWS environment with a companion [destroy command].
-->

<!--
>**Note:** The destroy command deletes all user-created structures in your AWS account, not just the ones created by `bosh aws create`.
-->

<!--
###Prepare a Domain
-->

<!--
1. Select a DNS domain name for your Cloud Foundry instance. For example, if you select the domain name cloud.example.com, Cloud Foundry deploys each of your applications as `APP-NAME.cloud.example.com`.
2. From the AWS [Route 53 dashboard], click `Hosted zones`.
3. Click the **Create Hosted Zone** button.
4. Under **Domain Name**, enter the domain name you chose above, for AWS to host.
5. Under **Type** choose **Public Hosted Zone**. This creates four name server (NS) records for your hosted zone.
6. From your domain registrar, delegate DNS authority for your hosted zone to your four Amazon Route 53 name servers. To do this, replace your registrar’s NS records for the domain with the NS record values you created in the last step.
-->

<!--
>**Note:** You can also find your NS record values by selecting your domain under the Hosted zones tab.
-->

![Hosted zones](../../images/deploy-clould-foundry/Preparing-to-Deploy-on-Amazon-Web-Services/hostedzone.png)

<!--
###Prepare the Deployment Environment
-->

<!--
1. Install the `bundler` RubyGem:

  ```shell
  $ gem install bundler
  ```
2. Create a `deployments` directory:

  ```shell
  $ mkdir deployments
  ```
3. From your `deployments` directory, make a sub-directory named `cf-example` to hold the gems and configuration information that BOSH needs to create your environment:

  ```shell
  $ cd deployments
  $ mkdir cf-example
  ```
4. In the `cf-example` sub-directory, create a file named `Gemfile` modeled after the following contents, but with the `ruby` argument matching the Ruby version that you are running:

  ```shell
  source 'https://rubygems.org'
  ruby "2.2.2"

  gem "bosh_cli_plugin_aws"
  ```
5. Run `bundle install` to install the `bosh_cli_plugin_aws` gem and its dependencies.

  ```shell
  $ bundle install

  ...

  Bundle complete! 1 Gemfile dependency, 85 gems now installed.
  Use `bundle show [gemname]` to see where a bundled gem is installed.
  ```
6. Generate a set of AWS access key credentials from the **Identity and Access Management** (IAM) Dashboard, under **Users**.
  * For a new user, click **Create New Users** and enter a user name.
  * For an existing user, choose the user, select the **Security Credentials** tab, and click **Create Access Key**.

7. Click **Show User Security Credentials** and record both the Access Key ID and Secret Access Key.

8. Make sure your user has permissions to create an environment for Cloud Foundry on AWS.
  * Select the **Permissions** tab on the user’s page.
  * Click **Attach Policy**.
  * Select the checkbox next to **AdministratorAccess**. This setting provides your user with full access to AWS services and resources.
  * Click **Attach Policy** at lower right.

  >**Note:** For more security, you can restrict the IAM user to a finer-grained set of permissions, but generally Administrator permissions will do.

9. Create a file named `bosh_environment` and add the following contents, replacing the values in each line to match your configuration as described below:

  ```shell
  export BOSH_VPC_DOMAIN=example.com
  export BOSH_VPC_SUBDOMAIN=my-subdomain
  export BOSH_AWS_ACCESS_KEY_ID=AWS_ACCESS_KEY_ID
  export BOSH_AWS_SECRET_ACCESS_KEY=AWS_SECRET_ACCESS_KEY
  export BOSH_AWS_REGION=us-east-1
  export BOSH_VPC_PRIMARY_AZ=us-east-1a
  export BOSH_VPC_SECONDARY_AZ=us-east-1b
  ```
  * The values for `BOSH_VPC_DOMAIN` and `BOSH_VPC_SUBDOMAIN` must correspond to the DNS domain name that you set up when [configuring Route 53]. This example uses `my-subdomain.example.com`.
  * For `BOSH_AWS_ACCESS_KEY_ID` and `BOSH_SECRET_ACCESS_KEY`, use the key pair you [created above] as AWS access key credentials.
  * For the `BOSH_AWS_REGION` property, use your AWS region. This example uses `us-east-1`.
  * For the `BOSH_VPC_PRIMARY_AZ` and `BOSH_VPC_SECONDARY_AZ` properties, choose an availability zone that is listed as “operating normally” in the Health Status section of the [AWS Console] for your region. This example uses `us-east-1a` and `us-east-1b`.
10. Run `source bosh_environment` to set the environment variables required for the AWS bootstrap tool, `bosh aws create`.
  ```shell
  $ source bosh_environment
  ```
11. Run `bosh aws create` to create a VPC Internet Gateway, VPC subnets, three RDS databases, and a NAT VM for Cloud Foundry subnet routing. This command generates two receipt files, `aws_rds_receipt.yml` and `aws_vpc_receipt.yml`, that you use when deploying Cloud Foundry.
  ```shell
  $ bosh aws create
  Executing migration CreateKeyPairs
  allocating 1 KeyPair(s)
  Executing migration CreateVpc

  ...

  details in S3 receipt: aws_rds_receipt and file: aws_rds_receipt.yml
  Executing migration CreateS3
  creating bucket xxxx-bosh-blobstore
  creating bucket xxxx-bosh-artifacts
  ```
  >**Note:** RDS database creation may take 20 or more minutes.
  >**Note:** If the `bosh aws create` command returns an error, you can destroy the AWS environment, as described in the next section.

12. From the AWS **EC2 Dashboard**, navigate to **Elastic IPs**.
13. Click **Allocate New Address** or select an existing Elastic IP that does not already list an associated instance.
14. With the IP address selected, associate it with the NAT VM instance by clicking **Actions** and selecting **Associate Address**.
15. In the **Associate Address** popup, click into the **Instance** field and select the instance listed as running `cf_nat_box1`.

  ![AWS Bind IP  to NAT](../../images/deploy-clould-foundry/Preparing-to-Deploy-on-Amazon-Web-Services/AWS-bind-IP-to-NAT.png)
-->

<!--
###Destroy the AWS Environment
-->

<!--
You can use `bosh aws destroy` to delete all user-created structures in your AWS environment. This is useful if `bosh aws create` does not complete successfully and you want to reset your environment, or if you want to tear down your AWS structures for any other reason.

```shell
$ bosh aws destroy
```
-->

<!--
This command prompts you to delete EC2 instances, EBS volumes, RDS databases, S3 buckets, VPCs, key pairs, Elastic IPs, security groups, and NS/SOA records in your account.
-->

<!--
>**WARNING:** The command `bosh aws destroy` destroys everything in your AWS account, including all S3 buckets and all instances. Do not use this command unless you want to lose **everything** in your AWS account, including objects and files unrelated to your Cloud Foundry deployment.
-->

[destroy command]: http://docs.cloudfoundry.org/deploying/aws/setup_aws.html#destroy-environment
[Route 53 dashboard]: https://console.aws.amazon.com/route53
[configuring Route 53]: http://docs.cloudfoundry.org/deploying/aws/setup_aws.html#domain-prep
[created above]: http://docs.cloudfoundry.org/deploying/aws/setup_aws.html#generate-credentials
[AWS Console]: https://console.aws.amazon.com/ec2/v2/home?region=us-east-1
