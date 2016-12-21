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

<!--
###Diego Architecture
-->

<!--
Cloud Foundry has used two architectures for managing application containers: [Droplet Execution Agents] (DEA) and Diego. With the DEA architecture, the [Cloud Controller] schedules and manages applications on the DEA nodes. In the newer Diego architecture, Diego components replace the DEAs and the [Health Manager (HM9000)], and assume application scheduling and management responsibility from the Cloud Controller.

Refer to the following diagram and descriptions for information about the way Diego handles application requests.
-->

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

<!--
###Diego Core Components
-->

<!--
Components in the Diego core run and monitor Tasks and LRPs. The core consists of the following major areas:

* [Brain]
* [Cells]
* [Database VMs]
* [Access VMs]
* [Consul]
-->

<!--
####Diego Brain
-->

<!--
Diego Brain components distribute Tasks and LRPs to Diego Cells, and correct discrepancies between `ActualLRP` and `DesiredLRP` counts to ensure fault-tolerance and long-term consistency. The Diego Brain consists of the Auctioneer.
-->

<!--
####Auctioneer
-->

<!--
Uses the [auction package] to run Diego Auctions for Tasks and LRPs
Communicates with Cell [Reps] over SSL/TLS
Maintains a lock in the BBS that restricts auctions to one Auctioneer at a time
Refer to the [Auctioneer repo] on GitHub for more information.
-->

<!--
####Diego Cell Components
-->

<!--
Diego Cell components manage and maintain Tasks and LRPs.
-->

<!--
#####Rep
-->

<!--
* Represents a Cell in Diego Auctions for Tasks and LRPs
* Mediates all communication between the Cell and the BBS
* Ensures synchronization between the set of Tasks and LRPs in the BBS with the containers present on the Cell
* Maintains the presence of the Cell in the BBS
* Runs Tasks and LRPs by asking the in-process Executor to create a container and RunAction recipes

Refer to the [Rep repo] on GitHub for more information.
-->

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

* Provides a consistent key-value data store to Diego

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

* Monitors `DesiredLRP` and `ActualLRP` states, emitting route registration and unregistration messages to the Cloud Foundry [router] when it detects changes
* Periodically emits the entire routing table to the Cloud Foundry router

Refer to the [Route-Emitter repo] on GitHub for more information.

[Droplet Execution Agents]: http://docs.cloudfoundry.org/concepts/architecture/execution-agent.html
[Cloud Controller]: http://docs.cloudfoundry.org/concepts/architecture/cloud-controller.html
[Health Manager (HM9000)]: http://docs.cloudfoundry.org/concepts/diego/dea-vs-diego.html#hm9k
[Diego Design Notes repo]: http://htmlpreview.github.io/?https://raw.githubusercontent.com/cloudfoundry-incubator/diego-design-notes/master/clickable-diego-overview/clickable-diego-overview.html
[Cloud Controller Bridge]: http://docs.cloudfoundry.org/concepts/diego/diego-architecture.html#bridge-components
[Tasks and Long Running Processes]: http://docs.cloudfoundry.org/concepts/diego/diego-auction.html#processes
[Bulletin Board System]: http://docs.cloudfoundry.org/concepts/diego/diego-architecture.html#bbs
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
