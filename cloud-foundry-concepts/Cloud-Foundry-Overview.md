<!--
Cloud Foundry Overview
Page last updated: December 7, 2016
-->
##Cloud Foundry概述

最后更新: 2016-12-7

<!--
###The Industry-Standard Cloud Platform
-->
###行业标准云平台

<!--
Cloud platforms let anyone deploy network apps or services and make them available to the world in a few minutes. When an app becomes popular, the cloud easily scales it to handle more traffic, replacing with a few keystrokes the build-out and migration efforts that once took months. Cloud platforms represent the next step in the evolution of IT, enabling you to focus exclusively on your applications and data without worrying about underlying infrastructure.
-->
云平台能够让任何人在几分钟之内就可以部署网络应用程序或者服务。当某个应用变得热门时，云平台能够轻松的扩容来应对更多的流量，以前需要数月才能完成的构建和迁移现在只需要简单的敲几下键盘就完成了。云平台代表了下一代IT革命，让你专门关注你的应用程序和数据而不用担心底层基础设施。

![power-of-platform](../images/cloud-foundry-concepts/power-of-platform.png)

<!--
Not all cloud platforms are created equal. Some have limited language and framework support, lack key app services, or restrict deployment to a single cloud. Cloud Foundry (CF) has become the industry standard. It is an [open source] platform that you can deploy to run your apps on your own computing infrastructure, or deploy on an IaaS like AWS, vSphere, or OpenStack. You can also use a PaaS deployed by a commercial [CF cloud provider]. A broad [community] contributes to and supports Cloud Foundry. The platform’s openness and extensibility prevent its users from being locked into a single framework, set of app services, or cloud.
-->
云平台功能各异。某些只支持部分语言和框架，缺少主要的服务，或者只能部署到某个云服务。Cloud Foundry(CF)已成为行业标准。它是一个完全[开源]的平台，你可以使用你自己的计算基础设施部署你的应用，或者使用IaaS服务例如AWS、vSphere或者OpenStack。你也可以使用现成的PaaS商业版的[CF云提供商]。

<!--
Cloud Foundry is ideal for anyone interested in removing the cost and complexity of configuring infrastructure for their apps. Developers can deploy their apps to Cloud Foundry using their existing tools and with zero modification to their code.
-->
Cloud Foundry是任何人想减少配置他们应用程序基础设施成本和复杂度较理想的工具。开发人员甚至可以使用他们已有的工具并且不用修改任何代码的情况下在Cloud Foundry部署他们的应用程序。

<!--
###How Cloud Foundry Works
-->
###Cloud Foundry如何工作

<!--
To flexibly serve and scale apps online, Cloud Foundry has subsystems that perform specialized functions. Here’s how some of these main subsystems work.
-->


<!--
####How the Cloud Balances Its Load
-->

<!--
Clouds balance their processing loads over multiple machines, optimizing for efficiency and resilience against point failure. A Cloud Foundry installation accomplishes this at three levels:

1. [BOSH] creates and deploys virtual machines (VMs) on top of a physical computing infrastructure, and deploys and runs Cloud Foundry on top of this cloud. To configure the deployment, BOSH follows a manifest document.
2. The CF [Cloud Controller] runs the apps and other processes on the cloud’s VMs, balancing demand and managing app lifecycles.
3. The [router] routes incoming traffic from the world to the VMs that are running the apps that the traffic demands, usually working with a customer-provided load balancer.
-->

<!--
####How Apps Run Anywhere
-->

<!--
Cloud Foundry designates two types of VMs: the component VMs that constitute the platform’s infrastructure, and the host VMs that host apps for the outside world. Within CF, the Diego system distributes the hosted app load over all of the host VMs, and keeps it running and balanced through demand surges, outages, or other changes. Diego accomplishes this through an auction algorithm.
-->

<!--
To meet demand, multiple host VMs run duplicate instances of the same app. This means that apps must be portable. Cloud Foundry distributes app source code to VMs with everything the VMs need to compile and run the apps locally. This includes the OS [stack] that the app runs on, and a [buildpack] containing all languages, libraries, and services that the app uses. Before sending an app to a VM, the Cloud Controller [stages] it for delivery by combining stack, buildpack, and source code into a droplet that the VM can unpack, compile, and run. For simple, standalone apps with no dynamic pointers, the droplet can contain a pre-compiled executable instead of source code, language, and libraries.
-->

<!--
How CF Organizes Users and Workspaces
-->

<!--
To organize user access to the cloud and to control resource use, a cloud operator defines [Orgs and Spaces] within an installation and assigns Roles such as admin, developer, or auditor to each user. The [User Authentication and Authorization] (UAA) server supports access control as an [OAuth2] service, and can store user information internally or connect to external user stores through LDAP or SAML.
-->

<!--
####Where CF Stores Resources
-->

<!--
Cloud Foundry uses the git system on [GitHub] to version-control source code, buildpacks, documentation, and other resources. Developers on the platform also use GitHub for their own apps, custom configurations, and other resources. To store large binary files, such as droplets, CF maintains an internal or external blobstore. To store and share temporary information, such as internal component states, CF uses MySQL, [Consul], and [etcd].
-->

<!--
####How CF Components Communicate
-->

<!--
Cloud Foundry components communicate with each other by posting messages internally using http and https protocols, and by sending [NATS] messages to each other directly.
-->

<!--
####How to Monitor and Analyze a CF Deployment
-->

<!--
As the cloud operates, the Cloud Controller VM, router VM, and all VMs running apps continuously generate logs and metrics. The Loggregator system aggregates this information in a structured, usable form, the Firehose. You can use all of the output of the Firehose, or direct the output to specific uses, such as monitoring system internals or analyzing user behavior, by applying nozzles.
-->

<!--
####Using Services with CF
-->

<!--
Typical apps depend on free or metered [services] such as databases or third-party APIs. To incorporate these into an app, a developer writes a Service Broker, an API that publishes to the Cloud Controller the ability to list service offerings, provision the service, and enable apps to make calls out to it.
-->

[开源]: https://github.com/cloudfoundry
[CF云提供商]: https://www.cloudfoundry.org/learn/certified-providers/
[community]: https://www.cloudfoundry.org/community/
[BOSH]: http://bosh.io/
[Cloud Controller]: http://docs.cloudfoundry.org/concepts/architecture/cloud-controller.html
[router]: http://docs.cloudfoundry.org/concepts/architecture/router.html
[stack]: http://docs.cloudfoundry.org/devguide/deploy-apps/stacks.html
[buildpack]: http://docs.cloudfoundry.org/buildpacks/
[stages]: http://docs.cloudfoundry.org/concepts/how-applications-are-staged.html
[Orgs and Spaces]: http://docs.cloudfoundry.org/concepts/roles.html
[User Authentication and Authorization]: http://docs.cloudfoundry.org/concepts/architecture/uaa.html
[OAuth2]: http://oauth.io/
[GitHub]: http://github.org/
[Consul]: https://github.com/hashicorp/consul
[etcd]: https://github.com/coreos/etcd
[NATS]: http://docs.cloudfoundry.org/concepts/architecture/messaging-nats.html
[services]: http://docs.cloudfoundry.org/services/overview.html
