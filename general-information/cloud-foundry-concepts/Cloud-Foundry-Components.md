<!--
##Cloud Foundry Components
Page last updated: December 9, 2016
-->
##Cloud Foundry组件

最后更新: 2016-12-9

<!--
Cloud Foundry components include a self-service application execution engine, an automation engine for application deployment and lifecycle management, and a scriptable command line interface (CLI), as well as integration with development tools to ease deployment processes. Cloud Foundry has an open architecture that includes a buildpack mechanism for adding frameworks, an application services interface, and a cloud provider interface.
-->
Cloud Foundry组件包括自助服务应用程序执行引擎，用于应用程序部署和生命周期管理的自动化引擎，以及可编写脚本的命令行接口（CLI），以及与开发工具集成以简化部署过程。 Cloud Foundry有一个开放式架构，包括用于自定义添加框架的buildpack机制，应用程序服务接口和云服务提供商接口。

<!--
Refer to the descriptions below for more information about Cloud Foundry components. Some descriptions include links to more detailed documentation.
-->
有关Cloud Foundry组件的更多信息，请参阅下面的说明。一些描述包括更详细的文档的链接。

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
[路由]分发请求到特定的组件，Cloud Foundry组件或运行在Diego Cell上的托管应用程序。

<!--
The router periodically queries the Diego Bulletin Board System (BBS) to determine which cells and containers each application currently runs on. Using this information, the router recomputes new routing tables based on the IP addresses of each cell virtual machine (VM) and the host-side port numbers for the cell’s containers.
-->
路由器定期查询Diego公告板系统（BBS），以确定每个应用程序当前运行的Cell和容器。使用此信息，路由器根据每个Cell虚拟机（VM）的IP地址和Cell容器的主机端端口号重新计算新的路由表。

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
[Cloud Controller]（CC）控制应用程序的部署。要将应用程序推送到Cloud Foundry，您需要使用位Cloud Foundry。Cloud Foundry然后指导Diego Brain通过[CC-Bridge]协调个别的[Diego cells]以暂存和运行应用程序。

在[Diego的前身架构]中，Cloud Foundry的Droplet执行代理（DEA）执行这些应用程序生命周期任务。

Cloud Foundry还维护[组织,空间,用户角色],[服务]等的记录。

<!--
####nsync, BBS, and Cell Reps
-->
####nsync, BBS, 和Cell Reps

<!--
To keep applications available, cloud deployments must constantly monitor their states and reconcile them with their expected states, starting and stopping processes as required. In pre-Diego architecture, the [Health Manager (HM9000)] performed this function. The nsync, BBS, and Cell Reps use a more distributed approach.
-->
为了保持应用程序可用，云部署必须持续监视其状态并根据其预期状态进行协调，按需启动和停止进程。在Diego的前身架构中，[Health Manager(HM9000)]执行同样的功能。 然而nsync，BBS和Cell Reps使用更分布式的方法。

![app-monitor-sync-diego](../../images/general-information/cloud-foundry-concepts/app-monitor-sync-diego.png)

<!--
The nsync, BBS, and Cell Rep components work together along a chain to keep apps running. At one end is the user. At the other end are the instances of applications running on widely-distributed VMs, which may crash or become unavailable.
-->
nsync, BBS, 和Cell Rep组件之间相互协作构成一条工作链共同保证应用程序正常运行。一端是用户，另一方面，在广泛分布的VM上运行的应用程序实例可能会崩溃或变得不可用。

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
blobstore是一个大二进制文件的存储库，Github无法轻松管理，因为Github是为代码而设计的。 Blobstore二进制文件包括：

<!--
* Application code packages
* Buildpacks
* Droplets
-->
* 应用程序代码包
* Buildpacks
* Droplets

<!--
You can configure the blobstore as either an internal server or an external S3 or S3-compatible endpoint.
-->
您可以将Blobstore配置为内部服务器或外部S3或S3兼容的终端。

<!--
####Diego Cell
-->
####Diego Cell

<!--
Each application VM has a Diego Cell that executes application start and stop actions locally, manages the VM’s containers, and reports app status and other data to the BBS and [Loggregator].
-->
每个应用程序都虚拟机有一个Diego Cell，它在本地执行应用程序启动和停止操作，管理VM的容器，并向BBS和[Loggregator]报告应用程序状态和其他数据。

<!--
In pre-Diego CF architecture, the [DEA node] performed the task of managing the applications and containers on a VM.
-->
在Diego早期架构中，[DEA节点]负责在虚拟机上管理应用程序和容器的任务。

<!--
###Services
-->
###服务

<!--
####Service Brokers
-->
####服务代理

<!--
Applications typically depend on [services] such as databases or third-party SaaS providers. When a developer provisions and binds a service to an application, the service broker for that service is responsible for providing the service instance.
-->
应用程序通常依赖某些[服务]，例如数据库或第三方SaaS提供程序。当某个开发人员创建并绑定某个服务到应用程序后，服务代理就会负责创建服务所需要的服务实例。

<!--
###Messaging
-->
###消息

<!--
####Consul and BBS
-->
####Consul和BBS

<!--
Cloud Foundry component VMs communicate with each other internally through HTTP and HTTPS protocols, sharing temporary messages and data stored in two locations:
-->
Cloud Foundry组件虚拟机通过HTTP和HTTPS协议在内部彼此通信，共享临时消息和存储在两个位置的数据：

<!--
* A [Consul server] stores longer-lived control data, such as component IP addresses and distributed locks that prevent components from duplicating actions.
* Diego’s [Bulletin Board System] (BBS) stores more frequently updated and disposable data such as cell and application status, unallocated work, and heartbeat messages. The BBS stores data in MySQL, using the [Go MySQL Driver].
-->
* [Consul服务器]存储较长生命周期的控制数据，例如组件IP地址和防止组件复制操作的分布式锁。
* Diego的[公告栏系统]（BBS）存储频繁更新或一次性的数据，例如执行单元和应用程序状态，未分配的工作和心跳测试消息。BBS使用[Go MySQL Driver]在MySQL中存储数据。

<!--
The route-emitter component uses the NATS protocol to broadcast the latest routing tables to the routers. In pre-Diego CF architecture, the [NATS Message Bus] carried all internal component communications.
-->
路由发射器组件使用NATS协议向路由器广播最新的路由表。在Diego前身架构中，[NATS消息总线]负责所有内部组件通信。

<!--
###Metrics and Logging
-->
###度量与日志

<!--
####Loggregator
-->
####Loggregator


<!--
The Loggregator (log aggregator) system streams application logs to developers.
-->
Loggregator(日志聚合器)系统将应用程序日志流式的传输给开发人员。

<!--
####Metrics Collector
-->
####度量收集器

<!--
The metrics collector gathers metrics and statistics from the components. Operators can use this information to monitor a Cloud Foundry deployment.
-->
度量收集器从组件收集度量和统计信息。操作员可以使用此信息来监控Cloud Foundry的部署。

[路由]: http://docs.cloudfoundry.org/concepts/architecture/router.html
[UAA]: http://docs.cloudfoundry.org/concepts/architecture/uaa.html
[Cloud Controller]: http://docs.cloudfoundry.org/concepts/architecture/cloud-controller.html
[CC-Bridge]: http://docs.cloudfoundry.org/concepts/diego/diego-architecture.html#bridge-components
[Diego cells]: http://docs.cloudfoundry.org/concepts/architecture/#diego-cell
[Diego的前身架构]: http://docs.cloudfoundry.org/concepts/diego/dea-vs-diego.html#design
[组织,空间,用户角色]: http://docs.cloudfoundry.org/concepts/roles.html
[服务]: http://docs.cloudfoundry.org/services/overview.html
[Health Manager(HM9000)]: http://docs.cloudfoundry.org/concepts/diego/dea-vs-diego.html#hm9k
[DEA节点]: http://docs.cloudfoundry.org/concepts/architecture/execution-agent.html
[服务]: http://docs.cloudfoundry.org/services/
[Consul服务器]: http://docs.cloudfoundry.org/concepts/diego/diego-architecture.html#consul
[公告栏系统]: http://docs.cloudfoundry.org/concepts/diego/diego-architecture.html#bbs
[Go MySQL Driver]: https://github.com/go-sql-driver/mysql
[NATS消息总线]: http://docs.cloudfoundry.org/concepts/diego/dea-vs-diego.html#nats
