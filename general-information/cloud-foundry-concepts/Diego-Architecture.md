<!--
##Diego Architecture

Page last updated: December 7, 2016
-->
###Diego架构

最后更新: 2016-12-07

<!--
This topic provides an overview of the structure and components of Diego, the new container management system for Cloud Foundry.

To deploy Diego, see the GitHub [Diego-Release].
-->
本主题概述了Cloud Foundry的新容器管理系统Diego的结构和组件。

要部署Diego，请参阅GitHub [Diego-Release]。

<!--
###Diego Architecture
-->
###Diego架构

<!--
Cloud Foundry has used two architectures for managing application containers: [Droplet Execution Agents] (DEA) and Diego. With the DEA architecture, the [Cloud Controller] schedules and manages applications on the DEA nodes. In the newer Diego architecture, Diego components replace the DEAs and the [Health Manager (HM9000)], and assume application scheduling and management responsibility from the Cloud Controller.

Refer to the following diagram and descriptions for information about the way Diego handles application requests.
-->

Cloud Foundry使用两种架构来管理应用程序容器：[Droplet Execution Agents](DEA)和Diego。使用DEA体系结构，[云控制器]在DEA节点上调度和管理应用程序。在较新的Diego结构中，Diego组件替换了DEA和[Health Manager(HM9000)]，并从云控制器承担应用程序调度和管理责任。

有关Diego处理应用程序请求方式的信息，请参阅以下图表和说明。

![diego-flow](../../images/general-information/cloud-foundry-concepts/diego-flow.png)

<!--
View a larger version of this image at the [Diego Design Notes repo].

1. The Cloud Controller passes requests to stage and run applications to the [Cloud Controller Bridge] (CC-Bridge).
2. The CC-Bridge translates staging and running requests into [Tasks and Long Running Processes] (LRPs), then submits these to the [Bulletin Board System] (BBS) through an API over HTTP.
3. The BBS submits the Tasks and LRPs to the [Auctioneer], part of the [Diego Brain].
4. The Auctioneer distributes these Tasks and LRPs to [Cells] through an [Auction]. The Diego Brain communicates with Diego Cells using SSL/TLS protocol.
5. Once the Auctioneer assigns a Task or LRP to a Cell, an in-process [Executor] creates a [Garden] container in the Cell. The Task or LRP runs in the container.
6. The [BBS] tracks desired LRPs, running LRP instances, and in-flight Tasks. It also periodically analyzes this information and corrects discrepancies to ensure consistency between `ActualLRP` and `DesiredLRP` counts.
7. The [Metron Agent], part of the Cell, forwards application logs, errors, and metrics to the Cloud Foundry Loggregator. For more information, see the [Application Logging in Cloud Foundry] topic.
-->
在[Diego设计说明]中查看该图片的放大版本。

1. 云控制器转发暂存和运行应用程序到请求到[Cloud Controller]（CC-Bridge）。
2. CC-Bridge将暂存和运行请求转换为[任务和长时间运行进程]（LRP），然后通过HTTP通过API将它们提交到[公告板系统]（BBS）。
3. BBS向[Auctioneer]提交任务和LRPs，[Auctioneer]是[Diego Brain]的一部分。
4. Auctioneer通过[Auction]将这些任务和LRP分发到[Cells]。 Diego Brain使用SSL/TLS协议与Diego Cells通信。
5. Auctioneer将任务或LRP分配给Cell后，进程[Executor]在单元中创建一个[Garden]容器。任务或LRP在容器中运行。
6. [BBS]跟踪期望的LRP，运行LRP实例和执行中的任务。它还定期分析此信息并更正差异，以确保`ActualLRP`和`DesiredLRP`计数之间的一致性。
7. [Metron Agent]是Cell的一部分，将应用程序日志，错误和指标转发到Cloud Foundry Loggregator。有关详细信息，请参阅[Cloud Foundry中的应用程序日志]主题。

<!--
###Diego Core Components
-->
###Diego核心组件

<!--
Components in the Diego core run and monitor Tasks and LRPs. The core consists of the following major areas:

* [Brain]
* [Cells]
* [Database VMs]
* [Access VMs]
* [Consul]
-->
Diego核心组件运行和监控着任务和LRPs。核心主要包括以下领域：

* [Brain]
* [Cells]
* [数据库虚拟机]
* [访问虚拟机]
* [Consul]

<!--
####Diego Brain
-->
####Diego Brain

<!--
Diego Brain components distribute Tasks and LRPs to Diego Cells, and correct discrepancies between `ActualLRP` and `DesiredLRP` counts to ensure fault-tolerance and long-term consistency. The Diego Brain consists of the Auctioneer.
-->
Diego Brain组件将任务和LRPs分发给Diego Cells，并且纠正`ActualLRP`和`DesiredLRP`之间的差异以确保容错和长久一致性。Diego Brain包括Auctioneer。

<!--
####Auctioneer
-->
####Auctioneer

<!--
* Uses the [auction package] to run Diego Auctions for Tasks and LRPs
* Communicates with Cell [Reps] over SSL/TLS
* Maintains a lock in the BBS that restricts auctions to one Auctioneer at a time

Refer to the [Auctioneer repo] on GitHub for more information.
-->
* 使用[auction package]运行Diego Auctions的任务和LRPs
* 通过SSL/TLS与Cell [Reps]通信
* 在BBS中维护一个锁，限制一次只执行一次Auctioneer

有关详细信息，请参阅GitHub上的[Auctioneer库]。

<!--
####Diego Cell Components
-->
####Diego Cell组件

<!--
Diego Cell components manage and maintain Tasks and LRPs.
-->
Diego Cell组件管理和维护任务和LRPs。

<!--
#####Rep
-->
#####Rep

<!--
* Represents a Cell in Diego Auctions for Tasks and LRPs
* Mediates all communication between the Cell and the BBS
* Ensures synchronization between the set of Tasks and LRPs in the BBS with the containers present on the Cell
* Maintains the presence of the Cell in the BBS
* Runs Tasks and LRPs by asking the to create a container and RunAction recipes

Refer to the [Rep repo] on GitHub for more information.
-->

* 表示任务和LRP的Diego Auctions的Cell
* Cell和BBS之间的所有通信的中介
* 确保BBS中的任务和LRPs的集合与Cell上存在的容器之间的同步
* 维护Cell在BBS中持续运行
* 通过请求进程内执行器创建容器和RunAction的方式来运行任务和LRPs

有关详细信息，请参阅GitHub上的[Rep库]。

<!--
#####Executor
-->
#####Executor

<!--
* Runs as a logical process inside the Rep
* Implements the generic Executor actions detailed in the [API documentation]
* Streams `STDOUT` and `STDERR` to the Metron agent running on the Cell

Refer to the [Executor repo] on GitHub for more information.
-->
* 在Rep中作为逻辑进程运行
* 实现[API文档]中详述的通用执行程序操作
* 将`STDOUT`和`STDERR`流输出到在Cell上运行的Metron Agent

有关详细信息，请参阅GitHub上的[Executor库]。

<!--
#####Garden
-->
#####Garden

<!--
* Provides a platform-independent server and clients to manage Garden containers
* Defines the [Garden-runC] interface for container implementation

See the [Garden] topic or the [Garden repository] on GitHub for more information.
-->
* 提供与平台无关的服务器和客户端来管理Garden容器
* 定义容器实现的[Garden-runC]接口

有关详细信息，请参阅[Garden]主题或GitHub上的[Garden库]。

<!--
#####Metron Agent
-->
#####Metron Agent

<!--
Forwards application logs, errors, and application and Diego metrics to the [Loggregator] Doppler component

Refer to the [Metron repo] on GitHub for more information.
-->
将应用程序日志，错误和应用程序以及Diego指标转发到[Loggregator] Doppler组件

有关详细信息，请参阅GitHub上的[Metron库]。

<!--
#####Database VMs
-->
#####数据库虚拟机

<!--
**Diego Bulletin Board System**

* Maintains a real-time representation of the state of the Diego cluster, including all desired LRPs, running LRP instances, and in-flight Tasks
* Provides an RPC-style API over HTTP to [Diego Core] components and external clients, including the [SSH Proxy], [CC-Bridge], and [Route Emitter].
* Ensure consistency and fault tolerance for Tasks and LRPs by comparing desired state (stored in the database) with actual state (from running instances)
* Acts to keep `DesiredLRP` count and `ActualLRP` count synchronized in the following ways:
  * If the `DesiredLRP` count exceeds the `ActualLRP` count, requests a start auction from the Auctioneer
  * If the `ActualLRP` count exceeds the `DesiredLRP` count, sends a stop message to the Rep on the Cell hosting an instance
* Monitors for potentially missed messages, resending them if necessary

Refer to the [Bulletin Board System repo] on GitHub for more information.
-->
** Diego公告栏系统**

* 维护Diego集群的实时状态，包括所有所需的LRPs，运行LRP实例和正在运行的任务
* 通过HTTP向[Diego Core]组件和外部客户端（包括[SSH代理]，[CC-Bridge]和[路由发射器]）提供RPC样式的API。
* 通过将期望的状态（存储在数据库中）与实际状态（来自正在运行的实例）做比较，确保任务和LRPs的一致性和容错性
* 以下列方式使`DesiredLRP`计数和`ActualLRP`计数保持同步：
  * 如果`DesiredLRP`计数超过`ActualLRP`计数，从Auctioneer发起请求
  * 如果`ActualLRP`计数超过`DesiredLRP`计数，发送停止消息到实例中Cell上的Rep
* 监视可能错过的消息，如有必要，重新发送

有关详细信息，请参阅GitHub上的[公告板系统资源库]。

<!--
#####MySQL
-->
#####MySQL

<!--
* Provides a consistent key-value data store to Diego
-->
* 为Diego提供一致的键值数据存储

<!--
####Access VMs
-->
####访问虚拟机

<!--
#####File Server
-->
#####文件服务器

<!--
* This “blobstore” serves static assets that can include general-purpose [App Lifecycle binaries] and application-specific droplets and build artifacts.

Refer to the [File Server repo] on GitHub for more information.
-->
* 此“blobstore”提供静态资产，可包括通用的[App Lifecycle二进制文件]和应用程序特定的droplets和构建产出物。

有关详细信息，请参阅GitHub上的[文件服务器]。

<!--
#####SSH Proxy
-->
#####SSH Proxy

<!--
* Brokers connections between SSH clients and SSH servers running inside instance containers

Refer to Understanding Application SSH, Application SSH Overview, or the Diego SSH Github repo for more information.
-->
* 在实例容器中运行的SSH客户端和SSH服务器之间的Brokers连接

有关详细信息，请参阅了[解应用程序SSH]，[应用程序SSH概述]或[Diego SSH Github库]。

<!--
#####Consul
-->
#####Consul

<!--
* Provides dynamic service registration and load balancing through DNS resolution
* Provides a consistent key-value store for maintenance of distributed locks and component presence

Refer to the Consul repo on GitHub for more information.
-->
* 通过DNS解析提供动态服务注册和负载均衡
* 为维护分布式锁和组件存在提供一致的键值存储

有关详细信息，请参阅GitHub上的[Consul库]。

<!--
#####Go MySQL Driver
-->
#####Go MySQL Driver

<!--
The Diego BBS stores data in MySQL. Diego uses the Go MySQL Driver to communicate with MySQL.

Refer to the [Go MySQL Driver repo] on GitHub for more information.
-->
Diego BBS在MySQL中存储数据。 Diego使用Go MySQL驱动程序与MySQL进行通信。

有关详细信息，请参阅GitHub上的[Go MySQL驱动程序库]。

<!--
###Cloud Controller Bridge Components
-->
###Cloud Controller桥接组件

<!--
The Cloud Controller Bridge (CC-Bridge) components translate app-specific requests from the Cloud Controller to the BBS. These components include the following:
-->
Cloud Controller桥接（CC-Bridge）组件将应用程序特定的请求从Cloud Controller转换到BBS。这些组件包括：

<!--
####Stager
-->
####Stager

<!--
* Translates staging requests from the Cloud Controller into generic Tasks and LRPs
* Sends a response to the Cloud Controller when a Task completes

Refer to the [Stager repo] on GitHub for more information.
-->
* 将来自Cloud Controller的暂存请求转换为通用任务和LRPs
* 任务完成后，向Cloud Controller发送响应

有关详细信息，请参阅GitHub上的[Stager库]。

<!--
####CC-Uploader
-->
####CC-Uploader

<!--
* Mediates uploads from the Executor to the Cloud Controller
* Translates simple HTTP POST requests from the Executor into complex multipart-form uploads for the Cloud Controller

Refer to the [CC-Uploader repo] on GitHub for more information.
-->
* 介入Executor到Cloud Controller之间
* 将来自Executor的简单HTTP POST请求转换为Cloud Controller的复杂多部分格式上传

有关详细信息，请参阅GitHub上的[CC-Uploader库]。

<!--
####Nsync
-->
####Nsync

<!--
* Listens for app requests to update the `DesiredLRPs` count and updates `DesiredLRPs` through the BBS
* Periodically polls the Cloud Controller for each app to ensure that Diego maintains accurate `DesiredLRPs` counts

Refer to the [Nsync repo] on GitHub for more information.
-->
* 监听更新`DesiredLRPs`计数的应用程序请求，并通过BBS更新`DesiredLRPs`
* 定期轮询每个应用程序的Cloud Controller，以确保Diego维护准确的`DesiredLRPs`计数

有关详细信息，请参阅GitHub上的[Nsync库]。

<!--
####TPS
-->
####TPS

<!--
* Provides the Cloud Controller with information about currently running LRPs to respond to `cf apps` and `cf app APP_NAME` requests
* Monitors `ActualLRP` activity for crashes and reports them the Cloud Controller

Refer to the [TPS repo] on GitHub for more information.
-->
* 为Cloud Controller提供有关当前运行的LRPs的信息，以响应`cf apps`和`cf app APP_NAME`请求
* 监视`ActualLRP`活动的崩溃，并向他们报告Cloud Controller

有关详细信息，请参阅GitHub上的[TPS库]。

<!--
###Platform-specific Components
-->
###平台特定组件

<!--
####Garden Backends
-->
####Garden后端

<!--
Garden contains a set of interfaces that each platform-specific backend must implement. See the [Garden] topic or the [Garden repository] on GitHub for more information.
-->
Garden包含每个特定于平台的后端必须实现的一组接口。有关详细信息，请参阅[Garden]主题或GitHub上的[Garden库]。

<!--
####App Lifecycle Binaries
-->
####App生命周期二进制文件

<!--
The following three platform-specific binaries deploy applications and govern their lifecycle:

* The **Builder**, which stages a CF application. The [CC-Bridge] runs the Builder as a Task on every staging request. The Builder performs static analysis on the application code and does any necessary pre-processing before the application is first run.
* The **Launcher**, which runs a CF application. The CC-Bridge sets the Launcher as the Action on the `DesiredLRP` for the application. The Launcher executes the start command with the correct system context, including working directory and environment variables.
* The **Healthcheck**, which performs a status check on running CF application from inside the container. The [CC-Bridge] sets the Healthcheck as the Monitor action on the `DesiredLRP` for the application.
-->

以下三个特定于平台的二进制文件部署应用程序并管理其生命周期：

* **构建器**，其中存储CF应用程序。 [CC-Bridge]在每个临时请求上将构建器作为任务运行。 构建器对应用程序代码执行静态分析，并在首次运行应用程序之前进行任何必要的预处理。
* **Launcher**，它运行一个CF应用程序。 CC-Bridge将Launcher设置为应用程序的`DesiredLRP`上的Action。 Launcher使用正确的系统上下文执行启动命令，包括工作目录和环境变量。
* **Healthcheck**，它从容器内部执行CF应用程序的状态检查。 [CC-Bridge]将Healthcheck设置为应用程序的`DesiredLRP`上的Monitor操作。

<!--
**Current Implementations**

* [Buildpack App Lifecycle] implements the Cloud Foundry buildpack-based deployment strategy.
* [Docker App Lifecycle] implements a Docker deployment strategy.
-->

**当前实现**

* [Buildpack应用程序生命周期]实现了Cloud Foundry基于buildpack的部署策略。
* [Docker应用程序生命周期]实现了Docker部署策略。

<!--
###Other Components
-->
###其他组件

<!--
####Route-Emitter
-->
####Route-Emitter

<!--
* Monitors `DesiredLRP` and `ActualLRP` states, emitting route registration and unregistration messages to the Cloud Foundry [router] when it detects changes
* Periodically emits the entire routing table to the Cloud Foundry router

Refer to the [Route-Emitter repo] on GitHub for more information.
-->
* 监视器`DesiredLRP`和`ActualLRP`状态，当它检测到更改时向Cloud Foundry[路由]发送路由注册和注销消息
* 定期将整个路由表发送到Cloud Foundry路由器

有关详细信息，请参阅GitHub上的[Route-Emitter库]。

[Diego-Release]: https://github.com/cloudfoundry-incubator/diego-release
[Droplet Execution Agents]: http://docs.cloudfoundry.org/concepts/architecture/execution-agent.html
[Cloud Controller]: http://docs.cloudfoundry.org/concepts/architecture/cloud-controller.html
[Health Manager(HM9000)]: http://docs.cloudfoundry.org/concepts/diego/dea-vs-diego.html#hm9k
[Diego设计说明]: http://htmlpreview.github.io/?https://raw.githubusercontent.com/cloudfoundry-incubator/diego-design-notes/master/clickable-diego-overview/clickable-diego-overview.html
[云控制器桥]: http://docs.cloudfoundry.org/concepts/diego/diego-architecture.html#bridge-components
[任务和长时间运行进程]: http://docs.cloudfoundry.org/concepts/diego/diego-auction.html#processes
[公告板系统]: http://docs.cloudfoundry.org/concepts/diego/diego-architecture.html#bbs
[Auctioneer]: http://docs.cloudfoundry.org/concepts/diego/diego-architecture.html#auctioneer
[Diego Brain]: http://docs.cloudfoundry.org/concepts/diego/diego-architecture.html#brain-components
[Cells]: http://docs.cloudfoundry.org/concepts/diego/diego-architecture.html#cell-components
[Auction]: http://docs.cloudfoundry.org/concepts/diego/diego-auction.html
[Executor]: http://docs.cloudfoundry.org/concepts/diego/diego-architecture.html#executor
[Garden]: http://docs.cloudfoundry.org/concepts/diego/diego-architecture.html#garden
[BBS]: http://docs.cloudfoundry.org/concepts/diego/diego-architecture.html#bbs
[Metron Agent]: http://docs.cloudfoundry.org/concepts/diego/diego-architecture.html#metron-agent
[Cloud Foundry中的应用程序日志]: http://docs.cloudfoundry.org/devguide/deploy-apps/streaming-logs.html
[Brain]: http://docs.cloudfoundry.org/concepts/diego/diego-architecture.html#brain-components
[Cells]: http://docs.cloudfoundry.org/concepts/diego/diego-architecture.html#cell-components
[数据库虚拟机]: http://docs.cloudfoundry.org/concepts/diego/diego-architecture.html#database-vms
[访问虚拟机]: http://docs.cloudfoundry.org/concepts/diego/diego-architecture.html#access-vms
[Consul]: http://docs.cloudfoundry.org/concepts/diego/diego-architecture.html#consul
[auction package]: https://github.com/cloudfoundry-incubator/auction
[Reps]: http://docs.cloudfoundry.org/concepts/diego/diego-architecture.html#rep
[Auctioneer库]: https://github.com/cloudfoundry-incubator/auctioneer
[Rep库]: https://github.com/cloudfoundry-incubator/rep
[API文档]: https://github.com/cloudfoundry-incubator/receptor/blob/master/doc/actions.md
[Executor库]: https://github.com/cloudfoundry-incubator/executor
[Garden-runC]: https://github.com/cloudfoundry/garden-runc-release
[Garden]: http://docs.cloudfoundry.org/concepts/architecture/garden.html
[Garden库]: https://github.com/cloudfoundry-incubator/garden
[Loggregator]: https://github.com/cloudfoundry/loggregator
[Metron库]: https://github.com/cloudfoundry/loggregator/tree/develop/src/metron
[Diego Core]: http://docs.cloudfoundry.org/concepts/diego/diego-architecture.html#core
[SSH代理]: http://docs.cloudfoundry.org/concepts/diego/diego-architecture.html#ssh-proxy
[CC-Bridge]: http://docs.cloudfoundry.org/concepts/diego/diego-architecture.html#bridge-components
[路由发射器]: http://docs.cloudfoundry.org/concepts/diego/diego-architecture.html#route-emitter
[公告板系统资源库]: https://github.com/cloudfoundry-incubator/bbs
[App Lifecycle二进制文件]: http://docs.cloudfoundry.org/concepts/diego/diego-architecture.html#app-lifecycles
[文件服务器]: https://github.com/cloudfoundry-incubator/file-server
[解应用程序SSH]: http://docs.cloudfoundry.org/concepts/diego/ssh-conceptual.html
[应用程序SSH概述]: http://docs.cloudfoundry.org/devguide/deploy-apps/app-ssh-overview.html
[Diego SSH Github库]: https://github.com/cloudfoundry-incubator/diego-ssh
[Consul库]: https://github.com/hashicorp/consul
[Go MySQL驱动程序库]: https://github.com/go-sql-driver/mysql
[Stager库]: https://github.com/cloudfoundry-incubator/stager
[CC-Uploader库]: https://github.com/cloudfoundry-incubator/cc-uploader
[Nsync库]: https://github.com/cloudfoundry-incubator/nsync
[TPS库]: https://github.com/cloudfoundry-incubator/tps
[Garden]: http://docs.cloudfoundry.org/concepts/architecture/garden.html
[Garden库]: https://github.com/cloudfoundry-incubator/garden
[CC-Bridge]: http://docs.cloudfoundry.org/concepts/diego/diego-architecture.html#bridge-components
[Buildpack应用程序生命周期]: https://github.com/cloudfoundry-incubator/buildpack-app-lifecycle
[Docker应用程序生命周期]: https://github.com/cloudfoundry-incubator/docker-app-lifecycle
[router]: https://github.com/cloudfoundry/gorouter
[Route-Emitter库]: https://github.com/cloudfoundry-incubator/route-emitter
