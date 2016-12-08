<!--
##Preparing to Deploy on Amazon Web Services
-->
##准备部署到亚马逊网络服务

<!--
The following steps document an example procedure for deploying Cloud Foundry on AWS. The process includes several steps that may be expensive and time-consuming, such as provisioning AWS Relational Database Service (RDS) instances. For a simpler deployment, refer to the Minimal [AWS] instructions.
-->
下面的步骤描述了一个在AWS上部署Cloud Foundry的例子。此过程包括几个步骤也许是一项开销很大且耗时的操作，例如配置AWS关系型数据库服务(RDS)实例。更简单的部署，可以参考AWS微型基础设施。

<!--
###Prerequisites
-->
###前提

<!--
To complete this deployment, you need:
-->
完成部署，你需要:

<!--
* An Amazon Web Services (AWS) account with Virtual Private Cloud (VPC)
* The BOSH CLI and bosh-init installed on your machine
* Ruby 1.9.3 or higher installed on your machine
* Sufficiently high instance limits on your AWS account. Installing Cloud Foundry using this bootstrap tool configures a production environment with more than 20 component VM instances.
-->
* [亚马逊网络服务(AWS)]账号和虚拟私有云(VPC)
* 安装[BOSH CLI]或者[bosh-init]
* [Ruby] 1.9.3或以上版本开发环境
* 保证你的AWS账户有足够的实例。安装Cloud Foundry并使用引导工具配置一个产品环境需要20多个组件虚拟实例。

<!--
>**Note**: You can deploy Cloud Foundry in smaller topologies, including a local development environment on Vagrant or a minimal AWS configuration.
-->
>**注意**: 你可以在较小的拓扑环境中部署Cloud Foundry，可以是安装Vagrant的[本地开发环境]或者是[AWS微型基础设施]。

<!--
###Step 1: Environment Setup
-->
###步骤 1: 环境设置

<!--
* [Setting up an AWS Environment for Cloud Foundry with BOSH AWS Bootstrap]: Use the BOSH AWS Bootstrap tool `bosh aws create` to create an environment on AWS that includes a VPC and other structures that support BOSH.
-->
* [使用BOSH AWS引导程序在AWS环境上配置Cloud Foundry]：使用BOSH AWS引导工具`bosh aws create`在AWS创建一个包含虚拟私有云(VPC)和其他BOSH需要的组件。

<!--
###Step 2: Deploy BOSH
-->
###步骤 2: 部署BOSH

<!--
* [Deploying BOSH on AWS]: Create a manifest for BOSH, and pass it to `bosh-init` to deploy BOSH in your environment.
-->
* [在AWS部署BOSH]：为BOSH创建一个部署清单，然后将它传给`bosh-init`在你的环境部署BOSH。

<!--
###Step 3: Deploy Cloud Foundry
-->
###步骤 3: 部署Cloud Foundry

<!--
* [Customizing the Cloud Foundry Deployment Manifest for AWS]
* [Deploying Cloud Foundry using BOSH]: Deploy!
-->
* [自定义AWS Cloud Foundry部署清单]
* [使用BOSH部署Cloud Foundry]： 部署!

[亚马逊网络服务(AWS)]: http://aws.amazon.com/
[BOSH CLI]: https://bosh.io/docs/bosh-cli.html
[bosh-init]: https://bosh.io/docs/install-bosh-init.html
[Ruby]: https://www.ruby-lang.org/en/documentation/installation/
[本地开发环境]: ./Preparing-to-Deploy-on-BOSH-Lite.md
[AWS微型基础设施]: https://github.com/cloudfoundry/cf-release/tree/master/example_manifests
[使用BOSH AWS引导程序在AWS环境上配置Cloud Foundry]: ./
[在AWS部署BOSH]: ./
[自定义AWS Cloud Foundry部署清单]: ./
[使用BOSH部署Cloud Foundry]: ./
