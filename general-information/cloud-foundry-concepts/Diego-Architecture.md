<!--
##Diego Architecture

Page last updated: December 7, 2016
-->
###Diego架构

最后更新: 2016-12-07

<!--
This topic provides an overview of the structure and components of Diego, the new container management system for Cloud Foundry.

To deploy Diego, see the GitHub Diego-Release.
-->
本主题概述了Cloud Foundry的新容器管理系统Diego的结构和组件。

要部署Diego，请参阅GitHub Diego-Release。

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
* [Database VMs]
* [Access VMs]
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

有关详细信息，请参阅GitHub上的[Auctioneer repo]。

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

有关详细信息，请参阅GitHub上的[Rep repo]。

<!--
#####Executor
-->

<!--
Runs as a logical process inside the Rep
Implements the generic Executor actions detailed in the [API documentation]
Streams `STDOUT` and `STDERR` to the Metron agent running on the Cell
Refer to the [Executor repo] on GitHub for more information.
-->

<!--
#####Garden
-->

<!--
Provides a platform-independent server and clients to manage Garden containers
Defines the [Garden-runC] interface for container implementation
See the [Garden] topic or the [Garden repository] on GitHub for more information.
-->

<!--
#####Metron Agent
-->

<!--
Forwards application logs, errors, and application and Diego metrics to the [Loggregator] Doppler component

Refer to the [Metron repo] on GitHub for more information.
-->

<!--
#####Database VMs
-->

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

<!--
#####MySQL
-->

<!--
* Provides a consistent key-value data store to Diego
-->

<!--
####Access VMs
-->

<!--
#####File Server
-->

<!--
* This “blobstore” serves static assets that can include general-purpose [App Lifecycle binaries] and application-specific droplets and build artifacts.

Refer to the [File Server repo] on GitHub for more information.
-->

<!--
#####SSH Proxy
-->

<!--
* Brokers connections between SSH clients and SSH servers running inside instance containers

Refer to Understanding Application SSH, Application SSH Overview, or the Diego SSH Github repo for more information.
-->

<!--
#####Consul
-->

<!--
* Provides dynamic service registration and load balancing through DNS resolution
* Provides a consistent key-value store for maintenance of distributed locks and component presence

Refer to the Consul repo on GitHub for more information.
-->

<!--
#####Go MySQL Driver
-->

<!--
The Diego BBS stores data in MySQL. Diego uses the Go MySQL Driver to communicate with MySQL.

Refer to the Go MySQL Driver repo on GitHub for more information.
-->

<!--
###Cloud Controller Bridge Components
-->

<!--
The Cloud Controller Bridge (CC-Bridge) components translate app-specific requests from the Cloud Controller to the BBS. These components include the following:
-->

<!--
####Stager
-->

<!--
* Translates staging requests from the Cloud Controller into generic Tasks and LRPs
* Sends a response to the Cloud Controller when a Task completes

Refer to the [Stager repo] on GitHub for more information.
-->

<!--
####CC-Uploader
-->

<!--
* Mediates uploads from the Executor to the Cloud Controller
* Translates simple HTTP POST requests from the Executor into complex multipart-form uploads for the Cloud Controller

Refer to the [CC-Uploader repo] on GitHub for more information.
-->

<!--
####Nsync
-->

<!--
* Listens for app requests to update the `DesiredLRPs` count and updates `DesiredLRPs` through the BBS
* Periodically polls the Cloud Controller for each app to ensure that Diego maintains accurate `DesiredLRPs` counts

Refer to the [Nsync repo] on GitHub for more information.
-->

<!--
####TPS
-->

<!--
* Provides the Cloud Controller with information about currently running LRPs to respond to `cf apps` and `cf app APP_NAME` requests
* Monitors `ActualLRP` activity for crashes and reports them the Cloud Controller

Refer to the [TPS repo] on GitHub for more information.
-->

<!--
###Platform-specific Components
-->

<!--
####Garden Backends
-->

<!--
Garden contains a set of interfaces that each platform-specific backend must implement. See the [Garden] topic or the [Garden repository] on GitHub for more information.
-->

<!--
####App Lifecycle Binaries
-->

<!--
The following three platform-specific binaries deploy applications and govern their lifecycle:

* The **Builder**, which stages a CF application. The [CC-Bridge] runs the Builder as a Task on every staging request. The Builder performs static analysis on the application code and does any necessary pre-processing before the application is first run.
* The **Launcher**, which runs a CF application. The CC-Bridge sets the Launcher as the Action on the `DesiredLRP` for the application. The Launcher executes the start command with the correct system context, including working directory and environment variables.
* The **Healthcheck**, which performs a status check on running CF application from inside the container. The [CC-Bridge] sets the Healthcheck as the Monitor action on the `DesiredLRP` for the application.


**Current Implementations**

* [Buildpack App Lifecycle] implements the Cloud Foundry buildpack-based deployment strategy.
* [Docker App Lifecycle] implements a Docker deployment strategy.

<!--
###Other Components
-->

<!--
####Route-Emitter
-->

<!--
* Monitors `DesiredLRP` and `ActualLRP` states, emitting route registration and unregistration messages to the Cloud Foundry [router] when it detects changes
* Periodically emits the entire routing table to the Cloud Foundry router

Refer to the [Route-Emitter repo] on GitHub for more information.
-->

[Droplet Execution Agents]: http://docs.cloudfoundry.org/concepts/architecture/execution-agent.html
[云控制器]: http://docs.cloudfoundry.org/concepts/architecture/cloud-controller.html
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
[Application Logging in Cloud Foundry]: http://docs.cloudfoundry.org/devguide/deploy-apps/streaming-logs.html
[Brain]: http://docs.cloudfoundry.org/concepts/diego/diego-architecture.html#brain-components
[Cells]: http://docs.cloudfoundry.org/concepts/diego/diego-architecture.html#cell-components
[Database VMs]: http://docs.cloudfoundry.org/concepts/diego/diego-architecture.html#database-vms
[Access VMs]: http://docs.cloudfoundry.org/concepts/diego/diego-architecture.html#access-vms
[Consul]: http://docs.cloudfoundry.org/concepts/diego/diego-architecture.html#consul
[auction package]: https://github.com/cloudfoundry-incubator/auction
[Reps]: http://docs.cloudfoundry.org/concepts/diego/diego-architecture.html#rep
[Auctioneer repo]: https://github.com/cloudfoundry-incubator/auctioneer
[Rep repo]: https://github.com/cloudfoundry-incubator/rep
[API documentation]: https://github.com/cloudfoundry-incubator/receptor/blob/master/doc/actions.md
[Executor repo]: https://github.com/cloudfoundry-incubator/executor
[Garden-runC]: https://github.com/cloudfoundry/garden-runc-release
[Garden]: http://docs.cloudfoundry.org/concepts/architecture/garden.html
[Garden repository]: https://github.com/cloudfoundry-incubator/garden
[Loggregator]: https://github.com/cloudfoundry/loggregator
[Metron repo]: https://github.com/cloudfoundry/loggregator/tree/develop/src/metron
[Diego Core]: http://docs.cloudfoundry.org/concepts/diego/diego-architecture.html#core
[SSH Proxy]: http://docs.cloudfoundry.org/concepts/diego/diego-architecture.html#ssh-proxy
[CC-Bridge]: http://docs.cloudfoundry.org/concepts/diego/diego-architecture.html#bridge-components
[Route Emitter]: http://docs.cloudfoundry.org/concepts/diego/diego-architecture.html#route-emitter
[Bulletin Board System repo]: https://github.com/cloudfoundry-incubator/bbs
[App Lifecycle binaries]: http://docs.cloudfoundry.org/concepts/diego/diego-architecture.html#app-lifecycles
[File Server repo]: https://github.com/cloudfoundry-incubator/file-server
[Understanding Application SSH]: http://docs.cloudfoundry.org/concepts/diego/ssh-conceptual.html
[Application SSH Overview]: http://docs.cloudfoundry.org/devguide/deploy-apps/app-ssh-overview.html
[Diego SSH Github repo]: https://github.com/cloudfoundry-incubator/diego-ssh
[Consul repo]: https://github.com/hashicorp/consul
[Go MySQL Driver repo]: https://github.com/go-sql-driver/mysql
[Stager repo]: https://github.com/cloudfoundry-incubator/stager
[CC-Uploader repo]: https://github.com/cloudfoundry-incubator/cc-uploader
[Nsync repo]: https://github.com/cloudfoundry-incubator/nsync
[TPS repo]: https://github.com/cloudfoundry-incubator/tps
[Garden]: http://docs.cloudfoundry.org/concepts/architecture/garden.html
[Garden repository]: https://github.com/cloudfoundry-incubator/garden
[CC-Bridge]: http://docs.cloudfoundry.org/concepts/diego/diego-architecture.html#bridge-components
[Buildpack App Lifecycle]: https://github.com/cloudfoundry-incubator/buildpack-app-lifecycle
[Docker App Lifecycle]: https://github.com/cloudfoundry-incubator/docker-app-lifecycle
[router]: https://github.com/cloudfoundry/gorouter
[Route-Emitter repo]: https://github.com/cloudfoundry-incubator/route-emitter
