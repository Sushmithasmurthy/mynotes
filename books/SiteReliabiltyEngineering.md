https://sre.google/books/ - My key takeaways by reading this book on SRE
## Introduction

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


## Sush - Continue from here - https://sre.google/sre-book/production-environment/
