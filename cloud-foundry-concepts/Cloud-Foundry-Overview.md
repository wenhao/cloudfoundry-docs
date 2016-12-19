<!--
Cloud Foundry Overview
Page last updated: December 7, 2016
-->
##Cloud Foundry概述
最后更新: 2016-12-7

<!--
The Industry-Standard Cloud Platform
-->

<!--
Cloud platforms let anyone deploy network apps or services and make them available to the world in a few minutes. When an app becomes popular, the cloud easily scales it to handle more traffic, replacing with a few keystrokes the build-out and migration efforts that once took months. Cloud platforms represent the next step in the evolution of IT, enabling you to focus exclusively on your applications and data without worrying about underlying infrastructure.
-->
![power-of-platform](../images/cloud-foundry-concepts/power-of-platform.png)

<!--
Not all cloud platforms are created equal. Some have limited language and framework support, lack key app services, or restrict deployment to a single cloud. Cloud Foundry (CF) has become the industry standard. It is an [open source] platform that you can deploy to run your apps on your own computing infrastructure, or deploy on an IaaS like AWS, vSphere, or OpenStack. You can also use a PaaS deployed by a commercial [CF cloud provider]. A broad [community] contributes to and supports Cloud Foundry. The platform’s openness and extensibility prevent its users from being locked into a single framework, set of app services, or cloud.
-->

<!--
Cloud Foundry is ideal for anyone interested in removing the cost and complexity of configuring infrastructure for their apps. Developers can deploy their apps to Cloud Foundry using their existing tools and with zero modification to their code.
-->

<!--
###How Cloud Foundry Works
-->

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

###开源PaaS是什么？
每一代计算机都会衍生出一种新的应用程序平台。在云计算领域，应用程序平台将被作为一种服务提供，通常被描述为平台即服务(PaaS)。PaaS使应用程序部署、运行和扩容更加容易。

不是所有的PaaS平台都一样，有些仅支持某些特定的语言或者框架，不能提供云应用程序所需要的主要应用服务，或者只能部署到某个云服务上。通过使用开源的PaaS平台，你可以选择适合你的云部署方案，开发的框架以及应用程序运行所需要的应用服务。此外，作为一个开源项目，背后有一个大型的社区在不断的改进并支持Cloud Foundry。

###Cloud Foundry是什么？

Cloud Foundry是一个开源的平台即服务，提供不同的云服务，开发框架及应用程序服务。Cloud Foundry让应用程序构建、测试、部署和扩容更加快捷和容易。它是一个[开源项目](https://github.com/cloudfoundry)，同时可以支持私有和共有云。

Cloud Foundry提供一个更为开放的平台及服务平台。大多数PaaS平台会限制开发框架、应用程序服务和部署的云服务。Cloud Foundry开放及可扩展的特性不会把开发人员限定在某个单一框架、应用服务及云服务之中。

###谁需要Cloud Foundry？

Cloud Foundry对于任何试图减少成本或者降低配置基础设施复杂度的开发人员是非常理想的。Cloud Foundry允许开发人员部署和扩容应用程序而不会只限定于某个云服务。开发人员可以通过已有的工具并且无需修改任何代码就可以轻松的部署他们的应用程序到Cloud Founddry上。

[开源]: https://github.com/cloudfoundry
[CF cloud provider]: https://www.cloudfoundry.org/learn/certified-providers/
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
