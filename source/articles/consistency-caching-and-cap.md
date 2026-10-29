# Consistency, caching and the CAP theorem

**Covers:** Strong consistency, Eventual consistency, CAP theorem, ACID, Cache

## What does "up to date" mean? {#overview}
Say you change your address on a website, and a friend loads your profile a second later. Do they see the new address? When data is stored in more than one place, that question has no automatic answer. Sometimes the answer must be yes, and sometimes "soon enough" is fine and much cheaper. In this article we will look at the options, at a theorem that sets limits on what is possible, and at the most common way to trade freshness for speed.

## ACID {#acid}
Start with a single database. ACID names four guarantees of reliable database transactions, where a transaction is a group of steps that must succeed or fail together:
- **Atomicity:** all steps happen, or none do.
- **Consistency:** the data always follows its rules.
- **Isolation:** transactions running at the same time do not disturb each other.
- **Durability:** once saved, the result survives a crash.

Moving money between two accounts is the classic case: subtracting from one and adding to the other must happen together. Traditional relational databases offer ACID transactions. Spreading data across many machines makes them harder, which is the starting point for the next ideas.

## Strong consistency {#strong-consistency}
With strong consistency, every reader sees the latest write immediately after it happens. If you change your address and then reload the page, you always see the new one, whichever server answers. It is the easiest to reason about, and the right choice for things like account balances and stock counts.

It costs time and availability. To guarantee it, the system may have to wait until all copies agree before replying, which is slower, and it may refuse to answer when it cannot reach some copies.

## Eventual consistency {#eventual-consistency}
With eventual consistency, readers will see the latest write eventually, but not necessarily right away. For a short time, different readers can see different values. If nothing else changes, all copies end up the same.

Think of a social network's like counter. If it shows 1,203 on your phone and 1,204 on mine for a second, nothing is harmed. In exchange, the system is faster and stays available, even during problems. It is a poor choice where a stale answer causes harm, such as selling the last ticket twice.

## CAP theorem {#cap-theorem}
The CAP theorem says that when a network failure splits a distributed system, it can fully guarantee only two of three properties: consistency (everyone sees the latest write), availability (every request gets an answer) and partition tolerance (it keeps working despite the split).

Network failures do happen, so partition tolerance is not optional. The real choice during a split is between consistency and availability. A system can refuse some requests to stay correct, or answer with possibly stale data to stay up. Outside a split, you can have both. The theorem is often over-simplified, but it gives you a useful question: "what should this system do when its parts cannot talk to each other?"

## Cache {#cache}
How do you make a slow lookup fast? You keep a copy somewhere quicker to reach than the original source. That copy is a cache. A shop might keep the result of an expensive product search for a minute. The next request is answered from the copy, quickly, and the database is spared.

The price is that the copy can become stale. If the product price changes, the cache may still show the old one until it expires or is cleared. Choosing how long to keep the data is a trade of freshness for speed. Caching is hard to get right, because deciding when a copy is out of date is the difficult part.

## A worked example {#example}
First, let's look at a replica that lags behind, which is eventual consistency.

```python
primary = {"stock": 5}
replica = {"stock": 5}

primary["stock"] = 4
print(replica["stock"])
replica.update(primary)
print(replica["stock"])
# 5
# 4
```

The write goes to the primary. A read from the replica straight afterwards still shows 5, which is stale. After the replication step, the replica shows 4. Between the two prints, the system was briefly inconsistent, and then it caught up.

Second, a cache that counts how often the slow work is done.

```python
cache = {}
calls = 0


def slow_square(n):
    global calls
    calls += 1
    return n * n


def cached_square(n):
    if n not in cache:
        cache[n] = slow_square(n)
    return cache[n]


for n in [3, 4, 3, 3, 4]:
    cached_square(n)
print(calls)
# 2
```

Five requests were made, but the slow function ran only twice, for 3 and for 4. The other three came from the cache. If each slow call took 100 milliseconds, the cache saved 3 x 100 = 300 milliseconds, which is 60 percent of the 500 milliseconds the work would otherwise take.

## A common mistake {#mistake}
A common slip is using eventual consistency for data where a stale read does harm, such as a balance or a seat count. Use strong consistency there.

A second slip is caching without an expiry or a way to clear stale entries, so users see old data indefinitely.

## Key points {#key-points}
- ACID transactions guarantee all-or-nothing, rule-following, isolated and durable changes.
- Strong consistency means every reader sees the latest write. Eventual consistency means they will, soon.
- The CAP theorem says that during a network split, you choose between consistency and availability.
- A cache trades freshness for speed. Plan how entries expire.
