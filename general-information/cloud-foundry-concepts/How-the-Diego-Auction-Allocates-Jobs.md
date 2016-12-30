<!--
##How the Diego Auction Allocates Jobs

Page last updated: December 2, 2016
-->

<!--
The [Diego] Auction balances application processes, also called jobs, over the virtual machines (VMs) in a Cloud Foundry installation. When new processes need to be allocated to VMs, the Diego Auction determines which ones should run on which machines. The auction algorithm balances the load on VMs and optimizes application availability and resilience. This topic explains how the Diego Auction works at a conceptual level.

The Diego Auction replaces the [Cloud Controller DEA placement algorithm], which performed the function of allocating processes to VMs in the pre-Diego Cloud Foundry architecture.

Refer to the [Auction repo] on GitHub for source code and more information.
-->

<!--
###Tasks and Long-Running Processes
-->

<!--
The Diego Auction distinguishes between two types of jobs: Tasks and Long-Running Processes (LRPs).
-->

<!--
* **Tasks** run once, for a finite amount of time. A common example is a staging task that compiles an app’s dependencies, to form a self-contained droplet that makes the app portable and runnable on multiple VMs. Other examples of tasks include making a database schema change, bulk importing data to initialize a database, and setting up a connected service.
* **Long-Running Processes** run continuously, for an indefinite amount of time. LRPs terminate only if stopped or killed, or if they crash. Examples include web servers, asynchronous background workers, and other applications and services that continuously accept and process input. To make high-demand LRPs more available, Diego may allocate multiple instances of the same application to run simultaneously on different VMs, often spread across Availability Zones that serve users in different geographic regions.
-->

<!--
The Diego Auction process repeats whenever new jobs need to be allocated to VMs. Each auction distributes a current **batch** of work, Tasks and LRPs, that can include newly-created jobs, jobs left unallocated in the previous auction, and jobs left orphaned by failed VMs. Diego does not redistribute jobs that are already running on VMs. Only one auction can take place at a time, which prevents placement collisions.
-->

<!--
###Ordering the Auction Batch
-->

<!--
The Diego Auction algorithm allocates jobs to VMs to fulfill the following outcomes, in decreasing **priority** order:
-->

<!--
1. Keep at least one instance of each LRP running.
2. Run all of the Tasks in the current batch.
3. Distribute as much of the total desired LRP load as possible over the remaining available VMs, by spreading multiple LRP instances broadly across VMs and their Availability Zones.
-->

<!--
To achieve these outcomes, each auction begins with the [Diego Auctioneer] component arranging the batch’s jobs into a priority order. Some of these jobs may be duplicate instances of the same process that Diego needs to allocate for high-traffic LRPs, to meet demand. So the Auctioneer creates a list of multiple LRP instances based on the desired instance count configured for each process.
-->

<!--
For example, if the process LRP-A has a desired instance count of 3 and a memory load of 2, and process LRP-B has 2 desired instances and a load of 5, the Auctioneer creates a list of jobs for each process as follows:
-->

| Process     | Desired Instances     | Load     | Jobs     |
| :------------- | :------------- | :------------- | :------------- |
| LRP-A       | 3       | ![diego-job-2][diego-job-2]       | ![diego-LRP-A-instances][diego-LRP-A-instances]       |
| LRP-B       | 2       | ![diego-job-5][diego-job-5]       | ![diego-LRP-B-instances][diego-LRP-B-instances]       |

<!--
The Auctioneer then builds an ordered sequence of LRP instances by cycling through the list of LRPs in decreasing order of load. With each cycle, it adds another instance of each LRP to the sequence, until all desired instances of the LRP have been added. With the example above, the Auctioneer would order the LRPs like this:
-->

![diego-LRP-stack][diego-LRP-stack]

<!--
The Auctioneer then builds an ordered sequence for all jobs, both LRPs and Tasks. Reflecting the auction batch [priority order], the first instances of LRPs are first priority. Tasks are next, in decreasing order of load. Duplicate LRP jobs come last.
-->

<!--
Adding one-time Task-C (load = 4) and Task-D (load = 3) to the above example, the priority order becomes:
-->
![diego-auction-stack][diego-auction-stack]

<!--
###Auctioning the Batch to the Cells
-->

<!--
With all jobs sorted in priority order, the Auctioneer allocates each in turn to one of the VMs. The process resembles an auction, where VMs “bid” with their suitability to run each job. Facilitating this process, each app VM has a resident [Cell] that monitors and allocates the machine’s operation. The Cell participates in the auction on behalf of the virtual machine that it runs on.
-->

<!--
Starting with the highest-priority job in the ordered sequence, the Auctioneer polls all the Cells on their fitness to run the currently-auctioned job. Cells “bid” to host each job according to the following priorities, in decreasing order:
-->

<!--
1. Allocate all jobs only to Cells that have the correct software stack to host them, and sufficient resources given their allocation so far during this auction.
2. Allocate LRP instances into Availability Zones that are not already hosting other instances of the same LRP.
3. Within each Availability Zone, allocate LRP instances to run on Cells that are not already hosting other instances of the same LRP.
4. Allocate any job to the Cell that has lightest load, from both the current auction and jobs it has been running already. In other words, distribute the total load evenly across all Cells.
-->

<!--
Our example auction sequence has seven jobs: five LRP instances and two Tasks. The following diagram shows how the Auctioneer might distribute this work across four Cells running in two Availability Zones:
-->
![diego-auction-process][diego-auction-process]

<!--
If the Auctioneer reaches the end of its sequence of jobs, having distributed all jobs to the Cells, it submits requests to the Cells to execute their allotted work. If the Cells ran out of capacity to handle all jobs in the sequence, the Auctioneer carries the unallocated jobs over and merges them into the next auction batch, to be allocated in the next auction.
-->

<!--
###Triggering Another Auction
-->

<!--
The Diego Auction process repeats to adapt a Cloud Foundry deployment to its changing workload. For example, the BBS initiates a new auction when it detects that the actual number of running instances of LRPs does not match the number desired. Diego’s BBS component monitors the number of instances of each LRP that are currently running. The [BBS] component periodically compares this number with the desired number of LRP instances, as configured by the user. If the actual number falls short of what is desired, the BBS triggers a new auction. In the case of a surplus of application instances, the BBS kills the extra instances and initiates another auction.
-->

<!--
The Cloud Controller also triggers an auction whenever a Cell fails. After any auction, if a Cell responds to its work request with a message that it cannot perform the work after all, the Auctioneer carries the unallocated work over into the next batch. But if the Cell fails to respond entirely, for example if its connection times out, the unresponsive Cell may still be running its work. In this case, the Auctioneer does not automatically carry the Cell’s work over to the next batch. Instead, the Auctioneer defers to the BBS to continue monitoring the states of the Cells, and to re-assign unassigned work later if needed.
-->

[Diego]: http://docs.cloudfoundry.org/concepts/diego/diego-architecture.html
[Cloud Controller DEA placement algorithm]: http://docs.cloudfoundry.org/concepts/architecture/dea-algorithm.html
[Auction repo]: https://github.com/cloudfoundry-incubator/auction
[Diego Auctioneer]: http://docs.cloudfoundry.org/concepts/diego/diego-architecture.html#auctioneer
[diego-job-2]: ../../images/general-information/cloud-foundry-concepts/diego-job-2.png
[diego-LRP-A-instances]: ../../images/general-information/cloud-foundry-concepts/diego-LRP-A-instances.png
[diego-job-5]: ../../images/general-information/cloud-foundry-concepts/diego-job-5.png
[diego-LRP-B-instances]: ../../images/general-information/cloud-foundry-concepts/diego-LRP-B-instances.png
[diego-LRP-stack]: ../../images/general-information/cloud-foundry-concepts/diego-LRP-stack.png
[priority order]: http://docs.cloudfoundry.org/concepts/diego/diego-auction.html#order
[diego-auction-stack]: ../../images/general-information/cloud-foundry-concepts/diego-auction-stack.png
[diego-auction-process]: ../../images/general-information/cloud-foundry-concepts/diego-auction-process.png
[Cell]: http://docs.cloudfoundry.org/concepts/diego/diego-architecture.html#cell-components
[BBS]: http://docs.cloudfoundry.org/concepts/diego/diego-architecture.html#bbs
