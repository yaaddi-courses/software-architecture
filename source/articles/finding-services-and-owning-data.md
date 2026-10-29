# Finding services and owning data

**Covers:** Service discovery, Database per service, Database-per-service, DB, DBs

## Two questions in a service-based system {#overview}
Once a system is split into services, two practical questions come up at once. How does one service find another, when machines come and go? And who owns the data: does each service keep its own, or do they all share? The answers shape how independent the services really are.

## DB {#db}
A DB, short for database, is an organized store of data that programs can read and write. It keeps records safely, lets many programs use them at once and answers queries quickly. Examples include PostgreSQL, MySQL and MongoDB. In design discussions, "the DB" often means the shared place where state lives, which makes it a likely point of tight coupling.

## Service discovery {#service-discovery}
Suppose the order service needs to call the payment service. Where is it? Addresses change all the time: services are restarted, moved, added when load rises and removed when it falls. Hard-coding "the payment service is at 10.0.0.5" breaks the first time that machine is replaced. Service discovery solves this by finding another service's current network address automatically, without a manual lookup.

The usual design has a registry, a list that services update. When a service starts, it registers its address, and when it stops, it is removed. A caller asks the registry "where is payments?" and receives the live addresses, often several, so it can spread calls among them. The registry checks health, so dead instances are dropped. Tools include Consul and etcd, and platforms such as Kubernetes offer it built in.

There are two styles. In client-side discovery, the caller asks the registry and picks an address. In server-side discovery, the caller sends the request to a router, which does the lookup.

## Database per service {#database-per-service}
Now the second question. In the database per service pattern, each service owns its data privately, and other services cannot read or write it directly. They must go through the service's API. This keeps the services independent: the orders team can change its tables, or switch to a different kind of database, without asking anyone.

The price is that you cannot join data across services with one query. If a screen needs orders and customer names, the system must call both services, or keep a copy. Keeping data consistent across services also becomes harder.

## Database-per-service {#database-per-service-hyphen}
You will see the same idea written with hyphens, as "database-per-service". It means exactly the same pattern. The opposite design, a shared database, lets many services use the same tables. That is easier at first, but it ties them together: a change to a table can break several services, and nobody can change it safely alone.

## A worked example {#example}
Two services need to cooperate. The first part shows discovery with a tiny registry.

```python
registry = {"payments": ["10.0.0.5:8080", "10.0.0.6:8080"]}


def find(name):
    return registry[name][0]


print(find("payments"))
registry["payments"].remove("10.0.0.5:8080")
print(find("payments"))
# 10.0.0.5:8080
# 10.0.0.6:8080
```

Let's read it:

1. The registry lists two copies of the payment service. The caller asks for "payments" and gets the first.
2. The first copy crashes, so its entry is removed from the registry.
3. The same call, with no change in the caller's code, now returns the second copy.

Now the data. A shop has an order service and a customer service. With a shared DB, the order service reads the `customers` table directly. When the customer team renames a column, orders stops working, and the break is found in production. With a database per service, the order service asks the customer service's API for the name. A column rename inside the customer service is invisible to orders.

The cost shows up in a report that needs both. A shared DB answers it with one join. With separate databases, the report makes two calls, or the orders service keeps a small copy of the customer's name that it updates when told of a change.

## A common mistake {#mistake}
A common slip is splitting into services but keeping one shared database. It looks like independence but is not, since every service is tied to the same tables.

A second slip is hard-coding addresses, which fails the first time a machine is replaced.

## Key points {#key-points}
- A DB is a shared store of data, and sharing one across services creates tight coupling.
- Service discovery lets services find each other's current addresses through a registry.
- Database per service means each service owns its data and exposes it only through its API.
- The price of separate databases is that joins and consistency need more work.
