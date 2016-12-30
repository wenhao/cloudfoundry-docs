<!--
##Differences Between DEA and Diego Architectures
-->
##DEA和Diego架构之间的区别

<!--
This topic describes components and functions that changed significantly when Cloud Foundry migrated to Diego architecture. This information will inform those who are familiar with Cloud Foundry’s DEA-based architecture and want to learn what has changed under Diego and how its new or changed components work.
-->
本主题描述了当Cloud Foundry迁移到Diego架构时变化的组件和功能。告知那些熟悉基于DEA架构的Cloud Foundry的人，并希望了解Diego有哪些变化，以及新的或变更的组件又是如何工作的。

<!--
###Key Differences
-->
###主要差异

<!--
The DEA architecture system is largely written in Ruby and the Diego architecture system is written in Go. When Cloud Foundry contributors decided to migrate the system’s core code from Ruby to Go, the rewrite offered the opportunity to make improvements to Cloud Foundry’s overall design.
-->
DEA架构系统主要用Ruby编写，Diego架构系统用Go编写。当Cloud Foundry贡献者决定将系统的核心代码从Ruby迁移到Go时，重写提供了改进Cloud Foundry的整体设计的机会。

<!--
In a pre-Diego Cloud Foundry deployment, the Cloud Controller’s [Droplet Execution Agent] (DEA) scheduled and managed applications on DEA nodes while the [Health Manager (HM9000)] kept them running. The Diego system assumes application scheduling and management responsibility from the Cloud Controller, replacing the DEA and Health Manager.
-->
在Diego之前的Cloud Foundry部署中，Cloud Foundry的[Droplet Execution Agent]（DEA）在DEA节点上安排和管理应用程序，而[Health Manager（HM9000）]保持它们运行。 Diego系统承担来自Cloud Foundry的应用程序调度和管理责任，取代了DEA和运行状况管理器。

<!--
DEA architecture made no distinction between machine jobs that run once and jobs that run continuously. Diego recognizes the difference and uses it to allocate jobs to virtual machines (VMs) more efficiently, replacing the [DEA Placement Algorithm] with the [Diego Auction].
-->
DEA架构对运行一次的机器作业和连续运行的作业没有区别。 Diego识别差异并使用它更有效地分配作业到虚拟机（VM），用[Diego Auction]替换[DEA替代算法]。

<!--
In addition to these broad changes, the Cloud Foundry migration to Diego architecture includes smaller changes and renamings. The following sections describe pre-Diego components and their newer analogs, and the [table] provides a summary.
-->
除了这些大的变化，Cloud Foundry迁移到Diego体系结构包括更小的更改和重命名。以下部分描述了Diego前身组件及新的组件，[表]提供了一个摘要。

<!--
###Changed Components and Functions
-->
###变化的组件和功能

<!--
####DEA Node → Diego Cell
-->
####DEA Node → Diego Cell

<!--
The pre-Diego [Droplet Execution Agent] (DEA) node component managed application instances, tracked started instances, and broadcast state messages on each application VM. These functions are now performed by the [Diego cell].
-->
Diego前身[Droplet Execution Agent]（DEA）节点组件在每个应用虚拟机上管理应用实例，跟踪启动实例和广播状态消息。这些功能现在由[Diego cell]执行。

<!--
####Warden → Garden
-->
####Warden → Garden

<!--
Pre-Diego application instances lived inside [Warden] containers, which are analogous to [Garden] containers in Diego architecture. Containerization ensures that application instances run in isolation, get their fair share of resources, and are protected from “noisy neighbors,” or other applications running on the same machine.
-->
Diego前身应用程序实例存在于[Warden]容器内，类似于Diego架构中的[Garden]容器。容器化确保应用程序实例独立运行，获得均等的资源共享，并且防止“嘈杂的邻居”或运行在同一台机器上的其他应用程序的干扰。

<!--
Warden could only manage containers on VMs running Linux, but the Garden subsystem supports VMs running diverse operating systems. The Garden front end presents the same container management operations that Warden used, with code that is abstracted away from any platform specifics. A platform-specific Garden Backend running on each VM translates the commands into machine code tailored to the native operating system.
-->
Warden只能管理运行Linux的虚拟机上的容器，但Garden子系统支持运行各种操作系统的虚拟机。 Garden前端提供了与Warden使用的相同的容器管理操作，代码可以从任何平台特性中抽象出来。在每个虚拟机上运行的特定于平台的Garden后端将命令转换为针对本机操作系统定制的机器代码。

<!--
The [Diego SSH package] enables developers to log into containers and access running application instances, a functionality that did not exist pre-Diego.
-->
[Diego SSH包]使开发人员能够登录到容器并访问正在运行的应用程序实例，这是在Diego前身不存在的功能。

<!--
**Warden Container-Level Traffic Rules**
-->
**Warden容器级流量规则**

<!--
For network security, pre-Diego releases of Cloud Foundry supported `allow` and `denyrules` that governed outbound traffic from all Warden containers running on the same DEA node. Newer releases use container-specific [Application Security Groups] (ASGs) to restrict traffic at a more granular level. Cloud Foundry recommends using ASGs exclusively, but when a pre-Diego deployment defined both Warden rules and ASGs, they were evaluated in a strict priority order.
-->
为了网络安全，Cloud Foundry的预发布版本支持`allow`和`denyrules`来管理在同一个DEA节点上运行的所有Warden容器的出站流量。较新的版本使用特定于容器的[应用程序安全组]（ASG）来限制更细粒度的流量。 Cloud Foundry建议专门使用ASG，但是在Diego前身的部署定义了Warden规则和ASG时，它们以严格的优先级顺序进行评估。

<!--
Pre-Diego Cloud Foundry returned an allow, deny, or reject result for the first rule that matched the outbound traffic request parameters, and did not evaluate any lower-priority rules. Cloud Foundry evaluated the network traffic rules for an application in the following order:
-->
Diego前身Cloud Foundry对与匹配出站流量请求参数的第一条规则返回允许，禁止或拒绝结果，并且未评估任何优先级较低的规则。 Cloud Foundry按照以下顺序评估应用程序的网络流量规则：

<!--
1. **Security Groups:** The rules described by the Default Staging set, the Default Running set, and all security groups bound to the space.
2. **Warden allow rules:** Any Warden Server configuration `allow` rules. Set Warden Server configuration rules in the Droplet Execution Agent (DEA) configuration section of your deployment manifest.
3. **Warden deny rules:** Any Warden Server configuration `deny` rules. Set Warden Server configuration rules in the DEA configuration section of your deployment manifest.
4. **Hard-coded reject rule:** Cloud Foundry returns a reject result for all outbound traffic from a container if not allowed by a higher-priority rule.
-->
1. **安全组:** 由默认配置集，默认运行集和绑定到该空间的所有安全组描述的规则。
2. **Warden允许规则:** 任何Warden服务器配置`allow`规则。在部署清单的Droplet Execution Agent（DEA）配置部分中设置Warden服务器配置规则。
3. **Warden deny规则:** 任何Warden服务器配置`deny`规则。在部署清单的DEA配置部分中设置Warden服务器配置规则。
4. **硬编码拒绝规则：** 如果较高优先级规则不允许的话，Cloud Foundry针对来自容器的所有出站流量返回拒绝结果。

<!--
####Health Manager (HM9000) → nsync, BBS, and Cell Rep
-->
####Health Manager (HM9000) → nsync, BBS, and Cell Rep

<!--
The function of the Health Manager (HM9000) component in pre-Diego releases of Cloud Foundry was replaced by the coordinated actions of the [nsync, BBS, and Cell Reps]. In pre-Diego architecture, the Health Manager (HM9000) had four core responsibilities:
-->
Cloud Foundry之前的Diego发行版中的健康管理器（HM9000）组件的功能被[nsync，BBS和Cell Reps]所取代。在前Diego架构中，健康管理（HM9000）有四个核心职责：

<!--
1. Monitor applications to determine their state (e.g. running, stopped, crashed, etc.), version, and number of instances. HM9000 updates the actual state of an application based on heartbeats and `droplet.exited` messages issued by the DEA node running the application.
2. Determine applications’ expected state, version, and number of instances. HM9000 obtains the desired state of an application from a dump of the Cloud Controller database.
3. Reconcile the actual state of applications with their expected state. For instance, if fewer than expected instances are running, HM9000 will instruct the Cloud Controller to start the appropriate number of instances.
4. Direct Cloud Controller to take action to correct any discrepancies in the state of applications.
-->
1. 监控应用程序以确定其状态（例如，运行，停止，崩溃等），版本和实例数。 HM9000基于运行应用程序的DEA节点发出的心跳和“droplet.exited”消息更新应用程序的实际状态。
2. 确定应用程序的预期状态，版本和实例数。 HM9000从转储Cloud Controller数据库获取应用程序的所需状态。
3. 将应用程序的实际状态与其预期状态进行协调。例如，如果运行少于预期的实例，HM9000将指示Cloud Foundry启动适当数量的实例。
4. 指示Cloud Controller采取措施纠正应用程序状态的任何差异。

<!--
HM9000 was essential to ensuring that apps running on Cloud Foundry remained available. HM9000 restarted applications whenever the DEA node running an app shut down for any reason, when Warden killed the app because it violated a quota, or when the application process exited with a non-zero exit code.

Refer to the [HM9000 readme] for more information about the HM9000 architecture.
-->
HM9000对于确保应用程序在Cloud Foundry上运行仍然起到至关重要的作用。每当运行应用程序的DEA节点不管什么原因关闭时，因违反配额Warden杀死该应用程序，或当应用程序进程非零退出时，HM9000都会重新启动应用程序

有关HM9000架构的更多信息，请参阅[HM9000自述文件]。

<!--
####DEA Placement Algorithm → Diego Auction
-->
####DEA Placement Algorithm → Diego Auction

<!--
In pre-Diego architecture, the Cloud Controller used the [DEA Placement Algorithm] to select the host DEA nodes for application instances that needed hosting.

Diego architecture moves this allocation process out of the Cloud Controller and into the Diego Brain, which uses the [Diego Auction] algorithm. The Diego Auction prioritizes one-time tasks like staging apps without affecting the uptime of ongoing, running applications like web servers.
-->
在前Diego架构中，Cloud Controller[DEA替换算法]为需要托管的应用程序实例选择主机DEA节点。

Diego架构将此分配过程从Cloud Controller移动到使用[Diego Auction]算法的Diego Brain中。 Diego Auction优先执行一次性任务，如暂存应用程序，而不影响正在运行的应用程序（如Web服务器）的正常运行时间。

<!--
####Message Bus (NATS)
-->
####Message Bus (NATS)

<!--
Pre-Diego Cloud Foundry used [NATS], a lightweight publish-subscribe and distributed queueing messaging system, for internal communication between components. Diego retains NATS for some communications, but adds messaging via HTTP and HTTPS protocols, through which components share information in the Consul and Diego BBS servers.
-->
前Diego Cloud Foundry使用[NATS]，一种轻量级发布订阅和分布式队列消息系统，用于组件之间的内部通信。 Diego为一些通信保留NATS，但通过HTTP和HTTPS协议发送消息，通过这些协议组件在Consul和Diego BBS服务器中共享信息。

<!--
###DEA / Diego Differences Summary
-->
###DEA / Diego差异摘要

<!--
| DEA architecture     | Diego architecture     | Function     | 	Δ notes    |
| :------------- | :------------- | :------------- | :------------- |
| Ruby       | Go       | Source code language       |        |
| [DEA]      | [Diego Brain]       | High-level coordinator that allocates processes to containers in application VMs and keeps them running       | DEA is part of the Cloud Controller. Diego is outside the Cloud Controller.       |
| DEA Node       | [Diego Cell]       | Mid-level manager on each VM that runs apps as directed and communicates “heartbeat”, application status and container location, and other messages       | Runs on each VM that hosts apps, as opposed to special-purpose component VMs.       |
| [Warden]       | Garden       | Low-level manager and API protocol on each VM for creating, configuring, destroying, monitoring, and addressing application containers       | Warden is Linux-only. Garden uses platform-specific Garden-backends to run on multiple OS.       |
| [DEA Placement Algorithm]       | [Diego Auction]       | Algorithm used to allocate processes to VMs       | Diego Auction distinguishes between Task and Long-Running Process (LRP) job types       |
| Health Manager (HM9000)       | [nSync, BBS, and Cell Reps]       | System that monitors application instances and keeps instance counts in sync with the number that should be running       | nSync syncs between Cloud Controller and Diego, BBS syncs within Diego, and Cell Reps sync between cells and the Diego BBS.       |
| NATS Message Bus       | [Bulletin Board System (BBS) and Consul] via http/s, and NATS       | Internal communication between components       | BBS stores most runtime data; Consul stores control data.  
-->
| DEA架构     | Diego架构     | 功能     | 	Δ 说明    |
| :------------- | :------------- | :------------- | :------------- |
| Ruby       | Go       | 开发语言      |        |
| [DEA]      | [Diego Brain] | 高级协调器，用于将进程分配给应用程序虚拟机中的容器，并使其保持运行       | DEA是Cloud Controller的一部分。 Diego在Cloud Controller之外。      |
| DEA节点       | [Diego Cell]       | 每个虚拟机上用于运行应用程序，发送“心跳”，应用程序状态和容器位置以及其他消息的中级管理器       | 在托管应用程序的每个虚拟机上运行，而不是专用组件虚拟机。       |
| [Warden]       | Garden       | 每个虚拟机上用于创建，配置，销毁，监控和寻址应用程序容器的低级管理器和API协议       |
warden只能使用Linux。 Garden使用平台特定的Garden-backends能在多个操作系统上运行。       |
| [DEA替代算法]       | [Diego Auction]       | 用于向虚拟机分配进程的算法       | Diego Auction区分任务和长时间运行进程（LRP）作业类型       |
| Health Manager (HM9000)       | [nSync，BBS和Cell Reps]       | 用来监控应用程序实例并使实例计数与应用程序实际运行的数字同步的系统       | nSync在Cloud Controller和Diego之间同步，BBS在Diego内同步，并且Cell Reps在Cell和Diego BBS之间同步。       |
| NATS消息总线       | [公告板系统(BBS)和Consul]使用 http/s和NATS       | 组件之间的内部通信       | BBS存储大多数运行时数据；Consul存储控制数据。  

[Droplet Execution Agent]: http://docs.cloudfoundry.org/concepts/architecture/execution-agent.html
[Health Manager (HM9000)]: http://docs.cloudfoundry.org/concepts/diego/dea-vs-diego.html#hm9k
[DEA替代算法]: http://docs.cloudfoundry.org/concepts/architecture/dea-algorithm.html
[Diego Auction]: http://docs.cloudfoundry.org/concepts/diego/diego-auction.html
[table]: http://docs.cloudfoundry.org/concepts/diego/dea-vs-diego.html#table
[Diego cell]: http://docs.cloudfoundry.org/concepts/architecture/#diego-cell
[Warden]: http://docs.cloudfoundry.org/concepts/architecture/warden.html
[Garden]: http://docs.cloudfoundry.org/concepts/diego/diego-architecture.html#cell-components
[Diego SSH包]: http://docs.cloudfoundry.org/concepts/diego/ssh-conceptual.html
[应用程序安全组]: http://docs.cloudfoundry.org/adminguide/app-sec-groups.html
[nSync，BBS和Cell Reps]: http://docs.cloudfoundry.org/concepts/architecture/#nsync-bbs
[HM9000自述文件]: https://github.com/cloudfoundry/hm9000
[NATS]: http://docs.cloudfoundry.org/concepts/architecture/messaging-nats.html
[DEA]: http://docs.cloudfoundry.org/concepts/architecture/execution-agent.html
[Diego Brain]: http://docs.cloudfoundry.org/concepts/diego/diego-architecture.html#brain-components
[公告板系统(BBS)和Consul]: http://docs.cloudfoundry.org/concepts/architecture/index.html#bbs-consul
