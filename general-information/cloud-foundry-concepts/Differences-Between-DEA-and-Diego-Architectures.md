<!--
##Differences Between DEA and Diego Architectures
-->

<!--
This topic describes components and functions that changed significantly when Cloud Foundry migrated to Diego architecture. This information will inform those who are familiar with Cloud Foundry’s DEA-based architecture and want to learn what has changed under Diego and how its new or changed components work.
-->

<!--
###Key Differences
-->

<!--
The DEA architecture system is largely written in Ruby and the Diego architecture system is written in Go. When Cloud Foundry contributors decided to migrate the system’s core code from Ruby to Go, the rewrite offered the opportunity to make improvements to Cloud Foundry’s overall design.

In a pre-Diego Cloud Foundry deployment, the Cloud Controller’s [Droplet Execution Agent] (DEA) scheduled and managed applications on DEA nodes while the [Health Manager (HM9000)] kept them running. The Diego system assumes application scheduling and management responsibility from the Cloud Controller, replacing the DEA and Health Manager.

DEA architecture made no distinction between machine jobs that run once and jobs that run continuously. Diego recognizes the difference and uses it to allocate jobs to virtual machines (VMs) more efficiently, replacing the [DEA Placement Algorithm] with the [Diego Auction].

In addition to these broad changes, the Cloud Foundry migration to Diego architecture includes smaller changes and renamings. The following sections describe pre-Diego components and their newer analogs, and the [table] provides a summary.
-->

<!--
###Changed Components and Functions
-->

<!--
####DEA Node → Diego Cell
-->

<!--
The pre-Diego [Droplet Execution Agent] (DEA) node component managed application instances, tracked started instances, and broadcast state messages on each application VM. These functions are now performed by the [Diego cell].
-->

<!--
####Warden → Garden
-->

<!--
Pre-Diego application instances lived inside [Warden] containers, which are analogous to [Garden] containers in Diego architecture. Containerization ensures that application instances run in isolation, get their fair share of resources, and are protected from “noisy neighbors,” or other applications running on the same machine.
-->

<!--
Warden could only manage containers on VMs running Linux, but the Garden subsystem supports VMs running diverse operating systems. The Garden front end presents the same container management operations that Warden used, with code that is abstracted away from any platform specifics. A platform-specific Garden Backend running on each VM translates the commands into machine code tailored to the native operating system.
-->

<!--
The [Diego SSH package] enables developers to log into containers and access running application instances, a functionality that did not exist pre-Diego.
-->

<!--
**Warden Container-Level Traffic Rules**
-->

<!--
For network security, pre-Diego releases of Cloud Foundry supported `allow` and `denyrules` that governed outbound traffic from all Warden containers running on the same DEA node. Newer releases use container-specific [Application Security Groups] (ASGs) to restrict traffic at a more granular level. Cloud Foundry recommends using ASGs exclusively, but when a pre-Diego deployment defined both Warden rules and ASGs, they were evaluated in a strict priority order.
-->

<!--
Pre-Diego Cloud Foundry returned an allow, deny, or reject result for the first rule that matched the outbound traffic request parameters, and did not evaluate any lower-priority rules. Cloud Foundry evaluated the network traffic rules for an application in the following order:
-->

<!--
1. **Security Groups:** The rules described by the Default Staging set, the Default Running set, and all security groups bound to the space.
2. **Warden allow rules:** Any Warden Server configuration `allow` rules. Set Warden Server configuration rules in the Droplet Execution Agent (DEA) configuration section of your deployment manifest.
3. **Warden deny rules:** Any Warden Server configuration `deny` rules. Set Warden Server configuration rules in the DEA configuration section of your deployment manifest.
4. **Hard-coded reject rule:** Cloud Foundry returns a reject result for all outbound traffic from a container if not allowed by a higher-priority rule.
-->

<!--
####Health Manager (HM9000) → nsync, BBS, and Cell Rep
-->

<!--
The function of the Health Manager (HM9000) component in pre-Diego releases of Cloud Foundry was replaced by the coordinated actions of the [nsync, BBS, and Cell Reps]. In pre-Diego architecture, the Health Manager (HM9000) had four core responsibilities:
-->

<!--
1. Monitor applications to determine their state (e.g. running, stopped, crashed, etc.), version, and number of instances. HM9000 updates the actual state of an application based on heartbeats and `droplet.exited` messages issued by the DEA node running the application.
2. Determine applications’ expected state, version, and number of instances. HM9000 obtains the desired state of an application from a dump of the Cloud Controller database.
3. Reconcile the actual state of applications with their expected state. For instance, if fewer than expected instances are running, HM9000 will instruct the Cloud Controller to start the appropriate number of instances.
4. Direct Cloud Controller to take action to correct any discrepancies in the state of applications.
-->

<!--
HM9000 was essential to ensuring that apps running on Cloud Foundry remained available. HM9000 restarted applications whenever the DEA node running an app shut down for any reason, when Warden killed the app because it violated a quota, or when the application process exited with a non-zero exit code.

Refer to the [HM9000 readme] for more information about the HM9000 architecture.
-->

<!--
####DEA Placement Algorithm → Diego Auction
-->

<!--
In pre-Diego architecture, the Cloud Controller used the [DEA Placement Algorithm] to select the host DEA nodes for application instances that needed hosting.

Diego architecture moves this allocation process out of the Cloud Controller and into the Diego Brain, which uses the [Diego Auction] algorithm. The Diego Auction prioritizes one-time tasks like staging apps without affecting the uptime of ongoing, running applications like web servers.
-->

<!--
####Message Bus (NATS)
-->

<!--
Pre-Diego Cloud Foundry used [NATS], a lightweight publish-subscribe and distributed queueing messaging system, for internal communication between components. Diego retains NATS for some communications, but adds messaging via HTTP and HTTPS protocols, through which components share information in the Consul and Diego BBS servers.
-->

<!--
###DEA / Diego Differences Summary
-->

| DEA architecture     | Diego architecture     | Function     | 	Δ notes    |
| :------------- | :------------- | :------------- | :------------- |
| Ruby       | Go       | Source code language       |        |
| [DEA]      | [Diego Brain]       | High-level coordinator that allocates processes to containers in application VMs and keeps them running       | DEA is part of the Cloud Controller. Diego is outside the Cloud Controller.       |
| DEA Node       | [Diego Cell]       | Mid-level manager on each VM that runs apps as directed and communicates “heartbeat”, application status and container location, and other messages       | Runs on each VM that hosts apps, as opposed to special-purpose component VMs.       |
| [Warden]       | Garden       | Low-level manager and API protocol on each VM for creating, configuring, destroying, monitoring, and addressing application containers       | Warden is Linux-only. Garden uses platform-specific Garden-backends to run on multiple OS.       |
| [DEA Placement Algorithm]       | [Diego Auction]       | Algorithm used to allocate processes to VMs       | Diego Auction distinguishes between Task and Long-Running Process (LRP) job types       |
| Health Manager (HM9000)       | [nSync, BBS, and Cell Reps]       | System that monitors application instances and keeps instance counts in sync with the number that should be running       | nSync syncs between Cloud Controller and Diego, BBS syncs within Diego, and Cell Reps sync between cells and the Diego BBS.       |
| NATS Message Bus       | [Bulletin Board System (BBS) and Consul] via http/s, and NATS       | Internal communication between components       | BBS stores most runtime data; Consul stores control data.       |

[Droplet Execution Agent]: http://docs.cloudfoundry.org/concepts/architecture/execution-agent.html
[Health Manager (HM9000)]: http://docs.cloudfoundry.org/concepts/diego/dea-vs-diego.html#hm9k
[DEA Placement Algorithm]: http://docs.cloudfoundry.org/concepts/architecture/dea-algorithm.html
[Diego Auction]: http://docs.cloudfoundry.org/concepts/diego/diego-auction.html
[table]: http://docs.cloudfoundry.org/concepts/diego/dea-vs-diego.html#table
[Diego cell]: http://docs.cloudfoundry.org/concepts/architecture/#diego-cell
[Warden]: http://docs.cloudfoundry.org/concepts/architecture/warden.html
[Garden]: http://docs.cloudfoundry.org/concepts/diego/diego-architecture.html#cell-components
[Diego SSH package]: http://docs.cloudfoundry.org/concepts/diego/ssh-conceptual.html
[Application Security Groups]: http://docs.cloudfoundry.org/adminguide/app-sec-groups.html
[nsync, BBS, and Cell Reps]: http://docs.cloudfoundry.org/concepts/architecture/#nsync-bbs
[HM9000 readme]: https://github.com/cloudfoundry/hm9000
[NATS]: http://docs.cloudfoundry.org/concepts/architecture/messaging-nats.html
[DEA]: http://docs.cloudfoundry.org/concepts/architecture/execution-agent.html
[Diego Brain]: http://docs.cloudfoundry.org/concepts/diego/diego-architecture.html#brain-components
[Bulletin Board System (BBS) and Consul]: http://docs.cloudfoundry.org/concepts/architecture/index.html#bbs-consul
