<!--
##Deploying BOSH on AWS
-->

<!--
In the [previous step] you prepared an AWS environment for BOSH. This document shows how to deploy BOSH into your AWS environment by creating a deployment manifest and passing it to `bosh-init`.
-->

<!--
###Copy the RSA Private Key
-->

<!--
When `bosh aws create` prepared your AWS environment, it generated an RSA private key file and saved it into your `~/.ssh` directory. To deploy BOSH, you need this file in your deployment directory.
-->

<!--
Copy the RSA private key file, `id_rsa_bosh`, to the `cf-example` directory you created in the [previous step].
-->

<!--
```shell
$ cp ~/.ssh/id_rsa_bosh ~/deployments/cf-example
```
-->

<!--
###Create a Deployment Manifest
-->

<!--
1. Log into the AWS Console.
2. Create a deployment manifest file named `bosh.yml` in the deployment directory. For its contents, copy the YAML code from the [AWS BOSH manifest template] in the BOSH documentation.
3. Replace properties listed in the file as follows:
  * `ELASTIC-IP`:In the **EC2 Dashboard** under **Elastic IPs**, use the Elastic IP with the Instance associated with **bosh**. Example: “203.0.113.126”.
  * `SUBNET-ID`: In the **VPC Dashboard** tab **Subnets**, use the Subnet ID for the subnet named “bosh1.” Example: “subnet-2f58134a”.
  * `AVAILABILITY-ZONE` and `REGION`: Use the values from the `bosh_environment` file you created in the [previous step].
  * `ACCESS-KEY-ID` and `SECRET-ACCESS-KEY`: Use the values from your `bosh_environment` file.
  * Replace all predefined passwords with passwords of your choice.
-->

<!--
###Deploy BOSH
-->

<!--
1. Confirm that [bosh-init] and the [BOSH CLI] are installed on your machine.
2. Run `bosh-init deploy ./bosh.yml` to start the deployment process.
  ```shell
  $ bosh-init deploy ./bosh.yml
  ...
  ```
  See `AWS CPI errors` for list of common errors and resolutions.
3. Use `bosh target ELASTIC-IP` to log into your new BOSH Director. The default username and password are `admin` and `admin`.
  ```shell
  $ bosh target 198.51.100.129

  Target set to 'bosh'
  Your username: admin
  Enter password: *****
  Logged in as 'admin'
  ```

For information about deploying BOSH on AWS without running `bosh aws create`, see the [Initializing BOSH environment on AWS] topic.
-->

[previous step]: ./Bootstrapping-an-AWS-Environment-for-Cloud-Foundry.md
[AWS BOSH manifest template]: AWS BOSH manifest template
[bosh-init]: http://bosh.cloudfoundry.org/docs/install-bosh-init.html
[BOSH CLI]: https://bosh.io/docs/bosh-cli.html
[Initializing BOSH environment on AWS]: http://bosh.io/docs/init-aws.html
