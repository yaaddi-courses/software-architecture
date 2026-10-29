# Scaling up, scaling out and balancing load

**Covers:** Horizontal scaling, Vertical scaling, CPU, RAM, Load balancer

## When one machine is not enough {#overview}
Every system has a limit on how much work one machine can do. When demand passes it, requests slow down and then fail. There are two basic ways to raise the limit, and a component that makes the second way work. First, a look at the resources that run out.

## CPU {#cpu}
The CPU, or central processing unit, is the processor that executes a program's instructions. A machine may have several cores, each able to run work in parallel. When all cores are busy, new work waits in line, so response times grow. Work that mostly calculates, such as image processing, is limited by the CPU.

## RAM {#ram}
RAM, or random-access memory, is the fast working memory in which a running program keeps its data. It is much faster than disk but smaller and lost when the power goes. When a program needs more RAM than the machine has, the system starts using the disk instead, which is far slower, or it stops the program. Work that holds a lot of data, such as large caches or big files in memory, is limited by RAM.

## Vertical scaling {#vertical-scaling}
Vertical scaling means handling more load by making one machine bigger, with more CPU cores, more RAM or faster disks. It is the simplest approach, since the software does not change. You restart on a larger machine and carry on.

It has limits. The biggest machine available has a ceiling, the price rises faster than the power and a single large machine is a single point of failure. It also usually needs a short outage to switch.

## Horizontal scaling {#horizontal-scaling}
Horizontal scaling means handling more load by adding more machines that share the work. When demand grows, you add copies of the service. When it drops, you remove them. There is no real ceiling, a failed machine costs only its share of the capacity and cloud platforms can add and remove machines automatically.

It requires the software to cooperate. Any copy must be able to handle any request, so state such as a user's session cannot live only in one machine's memory. It must be stored somewhere shared, such as a database or a cache. Data that must be consistent across machines makes this harder.

## Load balancer {#load-balancer}
A load balancer is a component that spreads incoming traffic across several servers, so that no single server is overloaded while others sit idle. It is the front door of a horizontally scaled service. Clients talk to one address, and the balancer forwards each request to one of the copies.

Common strategies: round robin sends requests to each server in turn, least connections favours the server with the fewest active requests, and weighted methods send more traffic to bigger machines. The balancer also checks the health of the servers and stops sending traffic to any that fail, so users do not notice. It must itself not be a single point of failure, so it is usually run in pairs.

## A worked example {#example}
A service handles 500 requests per second per server. Demand is 1,800 per second at peak.

```python
import math
from itertools import cycle

per_server = 500
demand = 1800
print(math.ceil(demand / per_server))
# 4

servers = cycle(["a", "b", "c"])
print([next(servers) for _ in range(7)])
# ['a', 'b', 'c', 'a', 'b', 'c', 'a']
```

How to read it:

1. 1,800 divided by 500 is 3.6, so at least 4 servers are needed. With exactly 4, each runs at 90 percent, which leaves little room, so a team would likely run 5.
2. The second part shows round robin: the balancer sends request 1 to `a`, 2 to `b`, 3 to `c`, then starts again.

Compare the two scaling styles for the same demand. Vertical: buy one machine that is 3.6 times as powerful, at a price that is often more than 3.6 times the cost, with a single point of failure. Horizontal: run 5 ordinary machines. If one fails, the other four still carry 4 x 500 = 2,000 requests per second, above the 1,800 demand, so users notice nothing.

Check the memory side too. If each request needs 50 MB of RAM at the same moment and a server has 16 GB, about 320 requests can be in flight at once on one server (16,000 divided by 50). If the load balancer lets 500 arrive, the server will run out of RAM before its CPU is full, so RAM, and not CPU, is the real limit.

## A common mistake {#mistake}
Scaling out before the software can handle it, for instance when each server keeps user sessions in its own memory. Users are then logged out when the balancer sends them to another copy. Another mistake is sizing for the average load. Plan for the peak, and for the loss of one server.

## Key points {#key-points}
- The CPU and RAM are the limited resources. Find out which one runs out first.
- Vertical scaling makes a machine bigger. It is simple but has a ceiling and a single point of failure.
- Horizontal scaling adds machines. It needs software that keeps no state in one machine.
- A load balancer spreads traffic and skips failed servers.
