https://sre.google/books/ - My key takeaways by reading this book on SRE
## Chapter 1 - Introduction

### Google’s Approach to Service Management: Site Reliability Engineering
once an SRE team is in place, their potentially unorthodox approaches to service management require strong management support. For example, the decision to stop releases for the remainder of the quarter once an error budget is depleted might not be embraced by a product development team unless mandated by their management.

### Tenets of SRE
In general, an SRE team is responsible for the availability, latency, performance, efficiency, change management, monitoring, emergency response, and capacity planning of their service(s).
The following section discusses each of the core tenets of Google SRE.

#### Ensuring a Durable Focus on Engineering
Google caps operational work for SREs at 50% of their time. Their remaining time should be spent using their coding skills on project work. In practice, this is accomplished by monitoring the amount of operational work being done by SREs, and redirecting excess operational work to the product development teams: reassigning bugs and tickets to development managers, [re]integrating developers into on-call pager rotations, and so on. The redirection ends when the operational load drops back to 50% or lower.
When they are focused on operations work, on average, SREs should receive a maximum of two events per 8–12-hour on-call shift. This target volume gives the on-call engineer enough time to handle the event accurately and quickly, clean up and restore normal service, and then conduct a postmortem.
Postmortems should be written for all significant incidents, regardless of whether or not they paged; postmortems that did not trigger a page are even more valuable, as they likely point to clear monitoring gaps. 

#### Pursuing Maximum Change Velocity Without Violating a Service’s SLO
Error budget:The error budget stems from the observation that 100% is the wrong reliability target for basically everything (pacemakers and anti-lock brakes being notable exceptions).the marginal difference between 99.999% and 100% gets lost in the noise of other unavailability, and the user receives no benefit from the enormous effort required to add that last 0.001% of availability.
If 100% is the wrong reliability target for a system, what, then, is the right reliability target for the system? This actually isn’t a technical question at all—it’s a product question, which should take the following considerations into account:

What level of availability will the users be happy with, given how they use the product?
What alternatives are available to users who are dissatisfied with the product’s availability?
What happens to users’ usage of the product at different availability levels?

An outage is no longer a "bad" thing—it is an expected part of the process of innovation, and an occurrence that both development and SRE teams manage rather than fear.

#### Monitoring
Monitoring should never require a human to interpret any part of the alerting domain. Instead, software should do the interpreting, and humans should be notified only when they need to take action.
There are three kinds of valid monitoring output:
##### Alerts
Signify that a human needs to take action immediately in response to something that is either happening or about to happen, in order to improve the situation.
##### Tickets
Signify that a human needs to take action, but not immediately. The system cannot automatically handle the situation, but if a human takes action in a few days, no damage will result.
##### Logging
No one needs to look at this information, but it is recorded for diagnostic or forensic purposes. The expectation is that no one reads logs unless something else prompts them to do so.

#### Emergency Response
Reliability is a function of mean time to failure (MTTF) and mean time to repair (MTTR) [Sch15]. The most relevant metric in evaluating the effectiveness of emergency response is how quickly the response team can bring the system back to health—that is, the MTTR.
Humans add latency. Even if a given system experiences more actual failures, a system that can avoid emergencies that require human intervention will have higher availability than a system that requires hands-on intervention

#### Change Management
Best practices in this domain **use automation** to accomplish the following:
* Implementing progressive rollouts
* Quickly and accurately detecting problems
* Rolling back changes safely when problems arise.

By **removing humans** from the loop, these practices avoid the normal problems of fatigue, familiarity/contempt, and inattention to highly repetitive tasks. As a result, both release velocity and safety increase.
As a result, both release velocity and safety increase.

#### Demand Forecasting and Capacity Planning
Several steps are mandatory in capacity planning:

* An accurate organic demand forecast, which extends beyond the lead time required for acquiring capacity
* An accurate incorporation of inorganic demand sources into the demand forecast
* Regular load testing of the system to correlate raw capacity(servers, disks, and so on) to service capacity

Because capacity is critical to availability, it naturally follows that the SRE team must be in charge of capacity planning, which means they also must be in charge of provisioning.

#### Provisioning
Provisioning combines both change management and capacity planning.

#### Efficiency and Performance
Resource use is a function of demand (load), capacity, and software efficiency. SREs predict demand, provision capacity, and can modify the software. These three factors are a large part (though not the entirety) of a service’s efficiency.


## Chapter - 2 - The Production Environment at Google, from the Viewpoint of an SRE
### Hardware
- Machine
  - A piece of hardware (or perhaps a VM)
- Server
  - A piece of software that implements a service
  
### System Software That "Organizes" the Hardware
- Managing Machines -
  - Borg, illustrated in Figure 2-2, is a distributed cluster operating system [Ver15], similar to Apache Mesos.10 Borg manages its jobs at the cluster level.
    - Borg is responsible for running users’ jobs, which can either be indefinitely running servers or batch processes like a MapReduce [Dea04].
    - Borg is also responsible for the allocation of resources to jobs
    - Borg allocates a name and index number to each task using the Borg Naming Service (BNS) - Rather than using the IP address and port number, other processes connect to Borg tasks via the BNS name, which is translated to an IP address and port number by BNS. For example, the BNS path might be a string such as /bns/<cluster>/<user>/<job name>/<task number>, which would resolve to <IP address>:<port>.
    - Borg can binpack the tasks over the machines in an optimal way that also accounts for failure domains
    - If a task tries to use more resources than it requested, Borg kills the task and restarts it (as a slowly crashlooping task is usually preferable to a task that hasn’t been restarted at all).
- Storage - 
  - The lowest layer is called D (for disk, although D uses both spinning disks and flash storage). D is a fileserver running on almost all machines in a cluster. However, users who want to access their data don’t want to have to remember which machine is storing their data, which is where the next layer comes into play.
  - A layer on top of D called Colossus creates a cluster-wide filesystem that offers usual filesystem semantics, as well as replication and encryption. Colossus is the successor to GFS, the Google File System
- Network - 
  - Instead of using "smart" routing hardware, we rely on less expensive "dumb" switching components in combination with a central (duplicated) controller that precomputes best paths across the network. Therefore, we’re able to move compute-expensive routing decisions away from the routers and use simple switching hardware.
  - Network bandwidth needs to be allocated wisely. Just as Borg limits the compute resources that a task can use, the Bandwidth Enforcer (BwE) manages the available bandwidth to maximize the average available bandwidth.

### Other System Software
- Lock Service
  - The Chubby [Bur06] lock service provides a filesystem-like API for maintaining locks. Chubby handles these locks across datacenter locations.
  - Chubby also plays an important role in master election. When a service has five replicas of a job running for reliability purposes but only one replica may perform actual work, Chubby is used to select which replica may proceed.
- Monitoring and Alerting
  - We want to make sure that all services are running as required. Therefore, we run many instances of our Borgmon monitoring program (see Practical Alerting from Time-Series Data). Borgmon regularly "scrapes" metrics from monitored servers.
    - Set up alerting for acute problems.
    - Compare behavior: did a software update make the server faster?
    - Examine how resource consumption behavior evolves over time, which is essential for capacity planning.

### Our Software Infrastructure
Data is transferred to and from an RPC using protocol buffers,12 often abbreviated to "protobufs," which are similar to Apache’s Thrift. Protocol buffers have many advantages over XML for serializing structured data: they are simpler to use, 3 to 10 times smaller, 20 to 100 times faster, and less ambiguous.

### Our Development Environment
- single shared repository
- Even large builds are executed quickly, as many build servers can compile in parallel
- Some projects use a push-on-green system, where a new version is automatically pushed to production after passing tests.

### Job and Data Organization
Load testing determined that our backend server can handle about 100 queries per second (QPS). Trials performed with a limited set of users lead us to expect a peak load of about 3,470 QPS, so we need at least 35 tasks. However, the following considerations mean that we need at least 37 tasks in the job, or N+2:
During updates, one task at a time will be unavailable, leaving 36 tasks.
A machine failure might occur during a task update, leaving only 35 tasks, just enough to serve peak load

## Sush to continue from here - https://sre.google/sre-book/part-II-principles/
