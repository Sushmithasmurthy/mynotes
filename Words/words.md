* antithetical - directly opposed or contrasted; mutually incompatible.
Example usage - 
The low-latency user wants Bigtable’s request queues to be (almost always) empty so that the system can process each outstanding request immediately upon arrival.
The user concerned with offline analysis is more interested in system throughput(number of requests served per unit time),
As you can see, success and failure are **antithetical** for these sets of users. Success for the low-latency user is failure for the user concerned with offline analysis.

