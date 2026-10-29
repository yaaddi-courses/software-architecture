# Microservices, SOA and serverless

**Covers:** Microservices, SOA, Serverless, AWS

## Splitting a system into services {#overview}
When one program grows too large for one team, a common answer is to split it into parts that run separately and talk over a network. There are several ways to do it, and they differ in how small the parts are and who runs the machines. In this article we describe three styles and the cloud platform that made one of them popular.

## SOA {#soa}
SOA, or service-oriented architecture, builds software as separate services that talk over a network. It became popular in the 2000s, mainly in large companies, with the idea that business functions such as "customer lookup" or "invoicing" could be reused by many applications.

Typical SOA services are fairly large and are connected through a central piece of software, often called an enterprise service bus, that routes and transforms messages. The approach brought reuse and a clear contract between teams. It also often brought heavy standards, slow change and a central bus that became a bottleneck.

## Microservices {#microservices}
Microservices are an architecture style in which a system is split into many small services, each of which can be built, deployed and scaled on its own. Each service has one focused job and usually owns its own data. Teams can work independently, choose suitable technology and release often.

The benefits are real: a busy service can be scaled without scaling the rest, a failure can be contained, and large organizations can organize teams around services. But the costs are real too. Calls that were function calls become network calls, which are slower and can fail. Tracking a single request across many services is harder. Data that used to be in one database is now spread out. The style needs mature automation and monitoring, and it suits large systems and teams more than small ones.

A common description of the difference from SOA: microservices are smaller, more independent, usually communicate through lightweight means and avoid a central bus.

## AWS {#aws}
AWS, or Amazon Web Services, is Amazon's cloud platform, where you rent computing power, storage, databases and many other services by the hour instead of buying machines. Launched in 2006, it made it cheap to start a service and to scale it up and down. Other large platforms include Microsoft Azure and Google Cloud.

AWS matters here because it provides the building blocks for service-based styles, from virtual servers to managed databases to the serverless functions described next.

## Serverless {#serverless}
In a serverless style, your code runs as short-lived functions that are triggered by events, and a cloud platform runs and scales them. You do not manage servers. You upload a function, say what triggers it, such as a web request or a new file, and pay only for the time it runs. AWS Lambda is a well-known example.

The benefits: no machines to patch, automatic scaling from zero to thousands, and no cost when idle. The trade-offs: a delay on the first call after a quiet period, limits on run time and memory, harder local testing and a risk of being tied to one provider. It suits spiky or occasional work, such as resizing uploaded images, and suits steady heavy workloads less well.

The name is misleading, since servers still exist. You do not run them, that is all.

## A worked example {#example}
A photo app must resize uploaded pictures. Let's compare three designs for 30,000 uploads a month, each taking 0.5 seconds of compute.

| Style | How it works | Notes |
|---|---|---|
| Microservices | A "resizer" service running on 2 always-on servers | Predictable. Costs the same when idle. Needs operating |
| SOA | The resizer is a large shared "media" service with other media features, reached through a central bus | Reusable but heavier to change |
| Serverless | A function triggered by each upload | Pays only for use |

Now the compute time for the serverless option: 30,000 uploads times 0.5 seconds is 15,000 seconds, about 4.2 hours of compute a month. A pair of always-on servers runs about 1,440 hours a month (2 servers times 24 hours times 30 days). Because the work is so occasional, the function runs for about 0.3 percent of that time (4.2 divided by 1,440), so paying per use costs far less.

If the load were steady at millions of uploads, the answer could reverse. The right style depends on the load and the team.

## A common mistake {#mistake}
A common slip is choosing microservices because large companies use them. Their reasons, many teams and huge scale, may not apply to you.

A second slip is calling a distributed tangle of services that all depend on each other "microservices". If services must be released together, they are a monolith in disguise.

## Key points {#key-points}
- SOA builds reusable, fairly large services, often linked by a central bus.
- Microservices are small, independently deployable services, with network and data costs.
- AWS is a cloud platform that provides the building blocks.
- Serverless runs functions on demand without you managing servers. It suits occasional work.
