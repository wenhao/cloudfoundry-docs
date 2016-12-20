<!--
##Cloud Foundry Components
Page last updated: December 9, 2016
-->
##Cloud Foundry组件

最后更新: 2016-12-9

<!--
Cloud Foundry components include a self-service application execution engine, an automation engine for application deployment and lifecycle management, and a scriptable command line interface (CLI), as well as integration with development tools to ease deployment processes. Cloud Foundry has an open architecture that includes a buildpack mechanism for adding frameworks, an application services interface, and a cloud provider interface.
-->
Cloud Foundry组件包括一个自我服务的应用程序运行引擎，一个自动化的应用程序部署引擎和生命周期管服务，还有一个脚本化的命令行接口(CLI)，同时也提供了一些开发工具来简化整个部署过程。Cloud Foundry整个系统拥有一个开发的架构，包括能够自定义添加框架的buildpack机制，开放的应用服务接口及云服务提供商接口。

<!--
Refer to the descriptions below for more information about Cloud Foundry components. Some descriptions include links to more detailed documentation.
-->
更多关于Cloud Foundry组件信息可以查看下图。某些描述信息还链接到更为详细的文档。

![Cloud Foundry架构](../../images/general-information/cloud-foundry-concepts/cf_architecture_block.png)

<!--
###Routing
-->
###路由

<!--
Router
-->
####路由器

<!--
The router routes incoming traffic to the appropriate component, either a Cloud Controller component or a hosted application running on a Diego Cell.
-->

[路由]分发请求到特定的组件，要么是控制器，要么是一个托管应用程序的Diego单元。

<!--
The router periodically queries the Diego Bulletin Board System (BBS) to determine which cells and containers each application currently runs on. Using this information, the router recomputes new routing tables based on the IP addresses of each cell virtual machine (VM) and the host-side port numbers for the cell’s containers.
-->
路由定期的查询Diego电子布告栏系统来决策哪个单元或者容器的应用程序在运行。通过这些信息，路由根据每个单元虚拟机的IP地址和容器的宿主端端口号重新计算路由表。

<!--
###Authentication
-->
###认证

<!--
####OAuth2 Server (UAA) and Login Server
-->
####OAuth2服务器(UUA)和登录服务器

<!--
The OAuth2 server (the UAA) and Login Server work together to provide identity management.
-->
OAuth2服务器([UAA])和登陆服务器共同管理认证服务。

<!--
###App Lifecycle
-->
###应用程序生命周期

<!--
####Cloud Controller and Diego Brain
-->
####控制器和Diego脑

<!--
The [Cloud Controller] (CC) directs the deployment of applications. To push an app to Cloud Foundry, you target the Cloud Controller. The Cloud Controller then directs the Diego Brain through the [CC-Bridge] to coordinate individual [Diego cells] to stage and run applications.

In [pre-Diego architecture], the Cloud Controller’s Droplet Execution Agent (DEA) performed these app lifecycle tasks.

The Cloud Controller also maintain records of [orgs, spaces, user roles, services], and more.
-->
[控制器](CC)控制应用程序的部署。为了部署应用程序到Cloud Foundry，你需要使用控制器。控制器通过[控制器桥]指导Diego脑协调每个[Diego单元]暂存和运行应用程序。

[Diego的前身架构]，控制器微小执行代理(DEA)管理应用程序的这些生命周期。

控制器也负责维护[组织,空间,用户角色],[服务]和其他事务。

<!--
####nsync, BBS, and Cell Reps
-->
####nsync, BBS, 和Cell Reps

<!--
To keep applications available, cloud deployments must constantly monitor their states and reconcile them with their expected states, starting and stopping processes as required. In pre-Diego architecture, the [Health Manager (HM9000)] performed this function. The nsync, BBS, and Cell Reps use a more distributed approach.
-->
为了保持应用程序持续运行，云部署必须持续不断的监控它们的状态并且使他们与期望的状态一致，按需启动或停止运行。Diego的前身架构中，[Health Manager(HM9000)]执行同样的功能。然而nsync, BBS, and Cell Reps使用分布式的方式。

![app-monitor-sync-diego](../../images/general-information/cloud-foundry-concepts/app-monitor-sync-diego.png)

<!--
The nsync, BBS, and Cell Rep components work together along a chain to keep apps running. At one end is the user. At the other end are the instances of applications running on widely-distributed VMs, which may crash or become unavailable.
-->
nsync, BBS, and Cell Rep组件之间相互协作构成一条工作链共同保证应用程序正常运行。一端是用户，另一端则是广阔的运行着的应用程序实例的虚拟机，虚拟机也有可能崩溃或者停止运行。

<!--
Here is how the components work together:

* **nsync** receives a message from the Cloud Controller when the user scales an app. It writes the number of instances into a `DesiredLRP` structure in the Diego BBS database.
* **BBS** uses its convergence process to monitor the `DesiredLRP` and `ActualLRP` values. It launches or kills application instances as appropriate to ensure the `ActualLRP` count matches the `DesiredLRP` count.
* **Cell Rep** monitors the containers and provides the `ActualLRP` value.
-->
以下是各组件如何工作的:

* **nsync** 当用户为应用程序扩容时，从控制器接受此动作消息。并把实例的数量以`DesiredLRP`的结构写入到Diego BBS数据库中。
* **BBS** 使用它的收敛程序比较`DesiredLRP`和`ActualLRP`的值。通过创建或者删除应用程序的实例来确保`ActualLRP`的数值和`DesiredLRP`的数据保持一致。
* **Cell Rep** 监控容器提供`ActualLRP`值。

<!--
###App Storage and Execution
-->
###应用程序的存储和执行

<!--
####Blobstore
-->
####Blobstore

<!--
The blobstore is a repository for large binary files, which Github cannot easily manage because Github is designed for code. Blobstore binaries include:
-->

<!--
* Application code packages
* Buildpacks
* Droplets
-->

<!--
You can configure the blobstore as either an internal server or an external S3 or S3-compatible endpoint.
-->

<!--
####Diego Cell
-->

<!--
Each application VM has a Diego Cell that executes application start and stop actions locally, manages the VM’s containers, and reports app status and other data to the BBS and [Loggregator].
-->

<!--
In pre-Diego CF architecture, the [DEA node] performed the task of managing the applications and containers on a VM.
-->

<!--
###Services
-->

<!--
####Service Brokers
-->

<!--
Applications typically depend on [services] such as databases or third-party SaaS providers. When a developer provisions and binds a service to an application, the service broker for that service is responsible for providing the service instance.
-->

<!--
###Messaging
-->

<!--
####Consul and BBS
-->

<!--
Cloud Foundry component VMs communicate with each other internally through HTTP and HTTPS protocols, sharing temporary messages and data stored in two locations:
-->

<!--
* A [Consul server] stores longer-lived control data, such as component IP addresses and distributed locks that prevent components from duplicating actions.
* Diego’s [Bulletin Board System] (BBS) stores more frequently updated and disposable data such as cell and application status, unallocated work, and heartbeat messages. The BBS stores data in MySQL, using the [Go MySQL Driver].
-->

<!--
The route-emitter component uses the NATS protocol to broadcast the latest routing tables to the routers. In pre-Diego CF architecture, the [NATS Message Bus] carried all internal component communications.
-->

<!--
###Metrics and Logging
-->

<!--
####Loggregator
-->

<!--
The Loggregator (log aggregator) system streams application logs to developers.
-->

<!--
####Metrics Collector
-->

<!--
The metrics collector gathers metrics and statistics from the components. Operators can use this information to monitor a Cloud Foundry deployment.
-->

###Cloud Controller

[Cloud Controller](http://docs.cloudfoundry.org/concepts/architecture/cloud-controller.html)负责管理应用程序的生命周期。当开发人员部署应用程序到Cloud Foundry后，应用程序就被Cloud Controller接管。Cloud Foundry会存储应用程序字节码文件，为应用程序元数据创建跟踪记录，然后分配DEA节点预处理和运行此应用程序。Cloud Foundry也维护组织、空间、服务、服务实例、用户角色等信息。

###HM9000

HM9000有四个主要的职责：

* 监控并获取应用程序的状态(例如：运行、停止和崩溃等), 版本号和实例数量。HM9000会根据运行在DEA上的应用程序的心跳测试和`droplet.exited`消息状态来更新应用程序的实际状态。
* 定义应用程序的预期状态，版本号和实例数量。HM9000可以从Cloud Controller数据库里查询应用程序的预期状态。
* 对比实际状态和预期状态，调整应用实例数量。举例来说，如果某个应用程序只有几个实例的运行状态是所期望的，HM9000就会通知Cloud Controller创建并启动对应数量的实例。
* 通知Cloud Controller处理任何状态错误的应用程序。

更多关于HM9000架构的详细信息参见[HM9000 readme](https://github.com/cloudfoundry/hm9000)。

###应用执行单元(DEA)

[Droplet 执行代理](http://docs.cloudfoundry.org/concepts/architecture/execution-agent.html)管理应用程序实例，跟踪任何已启动应用实例并广播应用程序各种状态信息。

应用程序实例运行在[Warden](http://docs.cloudfoundry.org/concepts/architecture/warden.html)容器内。集装箱式的架构保证应用程序实例之间相对独立，远离任何共享资源和阻碍其他实例干扰。

###大数据存储

大数据存储包括：

* 应用程序代码
* Buildpacks
* Droplets

###服务代理

应用程序通常依赖某些[服务](http://docs.cloudfoundry.org/services/)例如，数据库或者第三方SaaS提供商。当某个开发人员创建并绑定某个服务到应用程序后，服务代理就会负责创建服务所需要的服务实例。

###消息总线

Cloud Foundry使用[NATS](http://docs.cloudfoundry.org/concepts/architecture/messaging-nats.html)，一个轻量级的消息订阅和分发系统，用于组件之间的通信。

###日志与统计

度量收集器会收集各个组件的度量信息。管理员可以通过这些信息来监控Cloud Foundry的实例。

应用程序的日志聚集器会收集并发送应用程序的日志给开发人员。

[路由]: http://docs.cloudfoundry.org/concepts/architecture/router.html
[UAA]: http://docs.cloudfoundry.org/concepts/architecture/uaa.html
[控制器]: http://docs.cloudfoundry.org/concepts/architecture/cloud-controller.html
[控制器桥]: http://docs.cloudfoundry.org/concepts/diego/diego-architecture.html#bridge-components
[Diego单元]: http://docs.cloudfoundry.org/concepts/architecture/#diego-cell
[Diego的前身架构]: http://docs.cloudfoundry.org/concepts/diego/dea-vs-diego.html#design
[组织、空间、用户角色]: http://docs.cloudfoundry.org/concepts/roles.html
[服务]: http://docs.cloudfoundry.org/services/overview.html
[Health Manager(HM9000)]: http://docs.cloudfoundry.org/concepts/diego/dea-vs-diego.html#hm9k
