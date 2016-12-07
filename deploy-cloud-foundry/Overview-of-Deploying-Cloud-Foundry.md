<!--
##Overview of Deploying Cloud Foundry
-->
##部署Cloud Foundry概述

<!--
Cloud Foundry is designed to be configured, deployed, managed, scaled, and upgraded on any cloud IaaS provider. Cloud Foundry achieves this by leveraging [BOSH], an open source tool for release engineering, deployment, lifecycle management, and distributed systems monitoring.
-->
Cloud Foundry被设计成能够在任何IaaS(基础设施及服务)提供商上配置、部署、管理、扩展和升级。Cloud Foundry借助[BOSH]，一个开源的支持版本工程、部署、生命周期和分布式系统监控的工具来完成上述功能。

<!--
At a high level, the steps are the same regardless of IaaS:
-->
从高层面上说，不管是什么IaaS下列步骤都是一样的:

<!--
1. Set up all external dependencies, such as IaaS account, external load balancers, DNS records, and any additional components.
2. Create a manifest to deploy a BOSH Director.
3. Deploy the BOSH Director.
4. Create a manifest to deploy Cloud Foundry.
5. Deploy Cloud Foundry.
-->

1. 配置所有外部依赖，诸如IaaS账号、外部均衡服务器、DNS记录和任何其他的组件。
2. 创建一个BOSH主管的部署清单。
3. 部署BOSH主管。
4. 创建一个Cloud Foundry的部署清单。
5. 部署Cloud Foundry。

<!--
We also offer a quick `vagrant up` solution for steps 1-3 to let you focus on pushing apps or hacking on Cloud Foundry itself.
-->
我们也提供了使用`vagrant up`来快速完成步骤1-3的解决方案，你可以关注部署应用程序或者Cloud Foundry自身上。

<!--
Select one of the core supported infrastructures below to get started:
-->
从以下支持的基础设施服务中选择一个开始:

<!--
* AWS
* OpenStack
* vSphere
* vCloud Air or vCloud Director
  Or, if you just want the `vagrant up` experience, use BOSH-Lite:
* BOSH-Lite
-->
* AWS
* OpenStack
* vSphere
* vCloud Air or vCloud Director
  >Or, if you just want the `vagrant up` experience, use BOSH-Lite:
* BOSH-Lite

[BOSH]: https://bosh.io/
