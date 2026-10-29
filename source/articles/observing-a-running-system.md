# Observability: logs, metrics and traces

**Covers:** Observability, Logs, Metrics, Traces

## Knowing what a live system is doing {#overview}
When a system runs on many machines and you cannot attach a debugger, how do you find out why it is slow, or why one user's request failed? The answer is to make the system report on itself while it runs. This property is called observability, and it is built from three kinds of signals. Each answers a different question, and together they make problems findable.

## Observability {#observability}
Observability is the ability to understand what a running system is doing from the outside, by looking at the data it produces. A system is observable when you can ask a new question about its behavior, such as "why were requests slow for customers in Berlin at 3 p.m.?", and answer it without changing the code.

It is different from monitoring in one way. Monitoring watches for problems that you expected, using dashboards and alerts. Observability helps you investigate problems that you did not expect. The two work together. The raw material is three signals: logs, metrics and traces.

## Logs {#logs}
Logs are records of individual events that happened in a system, each a separate entry such as "user 42 logged in at 10:03" or "payment failed: card declined". They answer the question "what happened?" in detail.

Good logs are structured, which means each entry has named fields, such as the time, the level (info, warning, error), the request id and the user id, so a tool can search and filter them. They should not contain passwords or personal data. The weakness of logs is volume: a busy system produces millions of lines, which are costly to store and slow to search.

## Metrics {#metrics}
Metrics are numeric measurements tracked over time, such as requests per second, error rate, memory in use or the time requests take. They answer "how is the system doing, and is it getting worse?" They are small and cheap, so they can be kept for a long time and drawn as graphs.

A well-known checklist is the four golden signals: latency (how long requests take), traffic (how much demand), errors (how many requests fail) and saturation (how full the system is). Metrics tell you that something is wrong but rarely why.

## Traces {#traces}
Traces follow a single request as it travels across several services, recording the time spent in each step. They answer "where did the time go for this request?" Each step is called a span, and the spans of one request are linked by a shared trace id.

A trace of a checkout might show 40 milliseconds in the gateway, 60 in the order service, 800 in the payment service and 20 in the database. The slow step is visible at once. Traces are especially valuable in systems of many services, where no single log shows the whole journey.

## A worked example {#example}
These ten request times, in milliseconds, show why metrics need care.

```python
import statistics

latencies = [80, 90, 85, 95, 100, 88, 92, 1200, 90, 87]
print(statistics.mean(latencies))
# 200.7
print(statistics.median(latencies))
# 90.0
```

How to read it:

1. The average is 200.7 milliseconds, which looks like a slow service.
2. The median, the middle value, is 90, so a typical request is fast.
3. One request took 1,200, which pulls the average up. The average hides the fact that 9 of 10 requests are fine, and that one is very bad.

Now use all three signals for that bad request:
- The **metric** shows a spike in latency at 14:03 and in the error rate.
- The **trace** for the slow request shows 1,100 of its 1,200 milliseconds were spent calling the payment service.
- The **log** of the payment service for that trace id says "timeout talking to the card provider, retry 2 of 3".

Together they point to a cause in about five minutes: a slow card provider. With only the metric, the team would know that something was wrong, but would have to guess where.

## A common mistake {#mistake}
Collecting logs only and hoping to search them in a crisis. Without metrics you cannot see trends, and without traces you cannot follow a request across services. Another mistake is looking at averages. Use percentiles such as the 95th, which show how the slowest requests behave.

## Key points {#key-points}
- Observability is being able to understand a running system from the data it produces.
- Logs record events, metrics track numbers over time and traces follow a request across services.
- Averages can hide bad requests, so watch percentiles.
- Use all three signals together to find the cause of a problem.
