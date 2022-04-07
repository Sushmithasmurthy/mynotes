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

## Chapter 3 - Embracing Risk
Extreme reliability comes at a cost: maximizing stability limits how fast new features can be developed and how quickly products can be delivered to users, and dramatically increases their cost, which in turn reduces the numbers of features a team can afford to offer.
Put simply, a user on a 99% reliable smartphone cannot tell the difference between 99.99% and 99.999% service reliability! With this in mind, rather than simply maximizing uptime, Site Reliability Engineering seeks to balance the risk of unavailability with the goals of rapid innovation and efficient service operations, so that users’ overall happiness—with features, service, and performance—is optimized.
Site Reliability Engineering seeks to balance the risk of unavailability with the goals of rapid innovation and efficient service operations, so that users’ overall happiness—with features, service, and performance—is optimized.

### Managing Risk
cost does not increase linearly as reliability increments -an incremental improvement in reliability may cost 100x more than the previous increment. The costliness has two dimensions:
#### The cost of redundant machine/compute resources
Addition of redundant resources helps making systems more reliable at the same time it comes with additional cost.
#### The opportunity cost
engineers no longer work on new features and products for end users as they are focussed on features that diminish risk

### Measuring Service Risk

* For most services, the most straightforward way of representing risk tolerance is in terms of the acceptable level of unplanned downtime.
* Unplanned downtime is captured by the desired level of service availability, usually expressed in terms of the number of "nines"- 99.9%, 99.99%, or 99.999% availability.
* Each additional nine corresponds to an order of magnitude improvement toward 100% availability.

#### Time Based Availability
**Time Based Availability = uptime/ (uptime + downtime)**
Using this formula over the period of a year, we can calculate the acceptable number of minutes of downtime to reach a given number of nines of availability. For example, a system with an availability target of 99.99% can be down for up to 52.56 minutes in a year and stay within its availability target; see Availability Table for a table.

#### Aggregate availability
At Google, however, a time-based metric for availability is usually not meaningful because we are looking across globally distributed services. Our approach to fault isolation makes it very likely that we are serving at least a subset of traffic for a given service somewhere in the world at any given time (i.e., we are at least partially "up" at all times). Therefore, instead of using metrics around uptime, we define availability in terms of the request success rate. Aggregate availability shows how this yield-based metric is calculated over a rolling window (i.e., proportion of successful requests over a one-day window).
**Aggregate availability = successful requests/total requests**
For example, a system that serves 2.5M requests in a day with a daily availability target of 99.99% can serve up to 250 errors and still hit its target for that given day.

### Risk Tolerance of Services

#### Identifying the Risk Tolerance of Consumer Services

When a product team exists, that team is usually the best resource to discuss the reliability requirements for a service. In the absence of a dedicated product team, the engineers building the system often play this role either knowingly or unknowingly.

##### Types of failures
Which is worse for the service: a constant low rate of failures, or an occasional full-site outage? 
Both types of failure may result in the same absolute number of errors, but may have vastly different **impacts** on the business.

##### Cost
* If we were to operate and build these systems at one more nine of availability, what would our incremental rise in revenue be.
* Does this additional revenue offset the cost of reaching that level of availability


#### Identifying the Risk Tolerance of Infrastructure Services
##### Target level of availability
Consider Bigtable [Cha06], a massive-scale distributed storage system for structured data. Some consumer services serve data directly from Bigtable in the path of a user request. Such services need low latency and high reliability. Other teams use Bigtable as a repository for data that they use to perform offline analysis (e.g., MapReduce) on a regular basis. These teams tend to be more concerned about throughput than reliability. Risk tolerance for these two use cases is quite distinct.

One approach to meeting the needs of both use cases is to engineer all infrastructure services to be ultra-reliable. Given the fact that these infrastructure services also tend to aggregate huge amounts of resources, such an approach is usually far too expensive in practice

##### Types of failures
The low-latency user wants Bigtable’s request queues to be (almost always) empty so that the system can process each outstanding request immediately upon arrival.
The user concerned with offline analysis is more interested in system throughput(number of requests served per unit time),
As you can see, success and failure are **antithetical** for these sets of users. Success for the low-latency user is failure for the user concerned with offline analysis.

#### Cost
Partition infrastructure and offer it at multiple independent levels of service. In the Bigtable example, we can build two types of clusters: low-latency clusters and throughput clusters

The key strategy with regards to infrastructure is to deliver services with explicitly delineated levels of service thus enabling the clients to make the right risk and cost trade-offs when building their systems.

## Motivation for Error Budgets

* Product development performance is largely evaluated on product velocity, which creates an incentive to push new code as quickly as possible.
* SRE performance is (unsurprisingly) evaluated based upon reliability of a service, which implies an incentive to push back against a high rate of change.
(Indeed, Google SRE’s unofficial motto is "Hope is not a strategy.") Instead, our goal is to define an objective metric, agreed upon by both sides, that can be used to guide the negotiations in a reproducible way. The more data-based the decision can be, the better.
#### Software fault tolerance
  How hardened do we make the software to unexpected events? Too little, and we have a brittle, unusable product. Too much, and we have a product no one wants to use (but that runs very stably).
#### Testing
  Again, not enough testing and you have embarrassing outages, privacy data leaks, or a number of other press-worthy events. Too much testing, and you might lose your market.
#### Push frequency
  Every push is risky. How much should we work on reducing that risk, versus doing other work?
#### Canary duration and size
  It’s a best practice to test a new release on some small subset of a typical workload, a practice often called canarying. How long do we wait, and how big is the canary?

### Forming Your Error Budget
Our practice is then as follows:

Product Management defines an SLO, which sets an expectation of how much uptime the service should have per quarter.
The actual uptime is measured by a neutral third party: our monitoring system.
The difference between these two numbers is the "budget" of how much "unreliability" is remaining for the quarter.
As long as the uptime measured is above the SLO—in other words, as long as there is error budget remaining—new releases can be pushed.
For example, imagine that a service’s SLO is to successfully serve 99.999% of all queries per quarter. This means that the service’s error budget is a failure rate of 0.001% for a given quarter. If a problem causes us to fail 0.0002% of the expected queries for the quarter, the problem spends 20% of the service’s quarterly error budget.

### Benefits
The main benefit of an error budget is that it provides a common incentive that allows both product development and SRE to focus on finding the right balance between innovation and reliability.
Many products use this control loop to manage release velocity: as long as the system’s SLOs are met, releases can continue. If SLO violations occur frequently enough to expend the error budget, releases are temporarily halted while additional resources are invested in system testing and development to make the system more resilient, improve its performance, and so on. More subtle and effective approaches are available than this simple on/off technique:15 for instance, slowing down releases or rolling them back when the SLO-violation error budget is close to being used up.

## Chapter 4 - Service Level Objectives

Choosing appropriate metrics helps to drive the right action if something goes wrong, and also gives an SRE team confidence that a service is healthy.
* Service Level Indicators(SLI)
* Service Level Objectives(SLO)
* Service Level Agreements(SLA)

### Service Level Terminology

#### Indicators(SLI) 
An SLI is a service level indicator—a carefully defined quantitative measure of some aspect of the level of service that is provided.
* Most services consider the below as common SLIs
  * **request latency**—how long it takes to return a response to a request—as a key SLI.
  * **error rate**, often expressed as a fraction of all requests received, and 
  * **system throughput**, typically measured in requests per second. 
  * **availability** -fraction of the time that a service is usable. It is often defined in terms of the fraction of well-formed requests that succeed, sometimes called yield.
  * **Durability**—the likelihood that data will be retained over a long period of time—is equally important for data storage systems.
The measurements are often aggregated: i.e., raw data is collected over a measurement window and then turned into a rate, average, or percentile.availabilities of 99% and 99.999% can be referred to as "2 nines" and "5 nines" availability, respectively, and the current published target for Google Compute Engine availability is “three and a half nines”—99.95% availability.

#### Objectives(SLO)
An SLO is a service level objective: a target value or range of values for a service level that is measured by an SLI. A natural structure for SLOs is thus SLI ≤ target, or lower bound ≤ SLI ≤ upper bound. 

#### Agreements(SLA)
Finally, SLAs are service level agreements: an explicit or implicit contract with your users that includes consequences of meeting (or missing) the SLOs they contain. The consequences are most easily recognized when they are financial—a rebate or a penalty—but they can take other forms. An easy way to tell the difference between an SLO and an SLA is to ask "what happens if the SLOs aren’t met?": if there is no explicit consequence, then you are almost certainly looking at an SLO.
SRE doesn’t typically get involved in constructing SLAs, because SLAs are closely tied to business and product decisions. SRE does, however, get involved in helping to avoid triggering the consequences of missed SLOs. They can also help to define the SLIs: there obviously needs to be an objective way to measure the SLOs in the agreement, or disagreements will arise.

### Indicators in Practice

#### What Do You and Your Users Care About
Services tend to fall into a few broad categories in terms of the SLIs they find relevant:

* **User-facing serving systems,** such as the Shakespeare search frontends, generally care about availability, latency, and throughput. In other words: Could we respond to the request? How long did it take to respond? How many requests could be handled?
* **Storage systems** often emphasize latency, availability, and durability. In other words: How long does it take to read or write data? Can we access the data on demand? Is the data still there when we need it? See Data Integrity: What You Read Is What You Wrote for an extended discussion of these issues.
* **Big data systems**, such as data processing pipelines, tend to care about throughput and end-to-end latency. In other words: How much data is being processed? How long does it take the data to progress from ingestion to completion? (Some pipelines may also have targets for latency on individual processing stages.)
* **All systems** should care about correctness: was the right answer returned, the right data retrieved, the right analysis done? Correctness is important to track as an indicator of system health, even though it’s often a property of the data in the system rather than the infrastructure per se, and so usually not an SRE responsibility to meet.

#### Collecting Indicators
Many indicator metrics are most naturally gathered on the server side, using a monitoring system such as Borgmon (see Practical Alerting from Time-Series Data) or Prometheus, or with periodic log analysis—for instance, HTTP 500 responses as a fraction of all requests.

#### Aggregation
Most metrics are better thought of as **distributions** rather than averages.For example, for a latency SLI, some requests will be serviced quickly, while others will invariably take longer—sometimes much longer. A simple average can obscure these tail latencies, as well as changes in them. Figure 4-1 provides an example: although a typical request is served in about 50 ms, 5% of requests are 20 times slower! Monitoring and alerting based only on the average latency would show no change in behavior over the course of the day, when there are in fact significant changes in the tail latency (the topmost line).
User studies have shown that people typically prefer a slightly slower system to one with high variance in response time.
Using percentiles for indicators allows you to consider the shape of the distribution and its differing attributes: **a high-order percentile, such as the 99th or 99.9th, shows you a plausible worst-case value,** while using the 50th percentile (also known as the median) emphasizes the typical case. The higher the variance in response times, the more the typical user experience is affected by long-tail behavior, an effect exacerbated at high load by queuing effects. User studies have shown that people typically prefer a slightly slower system to one with high variance in response time, so some SRE teams focus only on high percentile values, on the grounds that if the 99.9th percentile behavior is good, then the typical experience is certainly going to be.
##### Note: 
Percentile Formula [[from thoughtco article](<https://www.thoughtco.com/what-is-a-percentile-3126238#:~:text=Percentiles%20can%20be%20calculated%20using,rank%20of%20a%20given%20value.&text=Percentiles%20are%20frequently%20used%20to%20understand%20test%20scores%20and%20biometric%20measurements>.)]

* Percentiles for the values in a given data set can be calculated using the formula: 
* `n = (P/100) x N`
* where N = number of values in the data set, P = percentile, and n = ordinal rank of a given value (with the values in the data set sorted from smallest to largest). For example, take a class of 20 students that earned the following scores on their most recent test: 75, 77, 78, 78, 80, 81, 81, 82, 83, 84, 84, 84, 85, 87, 87, 88, 88, 88, 89, 90. These scores can be represented as a data set with 20 values: {75, 77, 78, 78, 80, 81, 81, 82, 83, 84, 84, 84, 85, 87, 87, 88, 88, 88, 89, 90}.
* We can find the score that marks the 20th percentile by plugging in known values into the formula and solving for n:
* n = (20/100) x 20
* n = 4
* The fourth value in the data set is the score 78. This means that 78 marks the 20th percentile; of the students in the class, 20 percent earned a score of 78 or lower.

#### Standardize Indicators
We recommend that you standardize on common definitions for SLIs so that you don’t have to reason about them from first principles each time. Any feature that conforms to the standard definition templates can be omitted from the specification of an individual SLI, e.g.:
* Aggregation intervals: “Averaged over 1 minute”
* Aggregation regions: “All the tasks in a cluster”
* How frequently measurements are made: “Every 10 seconds”
* Which requests are included: “HTTP GETs from black-box monitoring jobs”
* How the data is acquired: “Through our monitoring, measured at the server”
* Data-access latency: “Time to last byte”
To save effort, build a set of reusable SLI templates for each common metric; these also make it simpler for everyone to understand what a specific SLI means.

#### Objectives in Practice
Start by thinking about (or finding out!) what your users care about, not what you can measure. Often, what your users care about is difficult or impossible to measure, so you’ll end up approximating users’ needs in some way.
working from desired objectives backward to specific indicators works better than choosing indicators and then coming up with targets.


## Chapter 5 Eliminating Toil

## Chapter 6 Monitoring Distributed Systems
start from here Chapter 6 - https://sre.google/sre-book/monitoring-distributed-systems/