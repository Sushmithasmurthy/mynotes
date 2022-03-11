* Extreme reliability comes at a cost 
* We should strive to make a service reliable enough, but no more reliable than it needs to be
* Inefficient queuing is often a cause if high tail latency
* Google SRE’s unofficial motto is "Hope is not a strategy."
* It’s the solution to managing tool sprawl -> For instance, App A might use an ADC from Vendor X while App B (which fails to comply with any security standards) might use the same ADC but a previous version of code that is insecure, or perhaps a totally different ADC from Vendor Y, and all of sudden you add tool sprawl to the mix too.