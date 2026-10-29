# CQRS and event sourcing

**Covers:** CQRS, CRUD, Event sourcing

## Two ways to rethink how data is stored and read {#overview}
Most applications keep one model of their data, and use it both to change the data and to show it. That works until the two jobs pull in different directions: writing needs strict rules and safety, while reading needs speed and convenient shapes. Two related ideas respond to this. One splits the reading and writing models. The other changes what is stored in the first place. Let's start with the one most applications begin with.

## CRUD {#crud}
CRUD is short for create, read, update and delete, the basic things you do with saved data. A CRUD application is one whose main job is to let users do exactly that, such as a contact list or a plain admin screen. It is a good fit when the rules are easy, and it is the starting point from which the other two ideas depart.

The weakness appears when the rules grow. A single model that must support validation, business rules, reports and fast screens becomes a compromise that serves none of them well.

## CQRS {#cqrs}
CQRS stands for command query responsibility segregation. The idea is to separate the model used to change data, the commands, from the model used to read it, the queries.

On the write side, commands such as "place order" are checked against the business rules and stored carefully. On the read side, queries use a model shaped for the screens that need it, such as a ready-made "orders per customer" summary, which can be stored in a different form, even in a different database. When something is written, the read side is updated.

The benefits: each side can be optimized and scaled separately, and reads can be very fast. The costs: more parts to build, and the read side may lag a little behind the write side. It suits systems with complex rules and heavy reading, and not simple CRUD applications.

## Event sourcing {#event-sourcing}
Event sourcing changes what you store. Instead of keeping only the current state, you store every change as an event. A bank account is the classic example. Rather than storing "balance: 120", you store the list: deposited 100, withdrew 30, deposited 50. The balance is worked out by replaying the list.

Events are never changed or deleted, only added. This gives a complete history for free: you can answer "what was the balance last Tuesday?", audit every change, find the cause of a bug by replaying and build new views from old events. The costs are that the model is less familiar, that replaying long histories can be slow unless you save snapshots, and that correcting an old mistake means adding a correcting event instead of editing.

CQRS and event sourcing are often combined: the events are the write side, and read models are built from them.

## A worked example {#example}
The first program keeps the history as events and replays it. The second keeps a ready-made read model next to the write side, the CQRS idea.

```python
events = [("deposit", 100), ("withdraw", 30), ("deposit", 50)]

balance = 0
for kind, amount in events:
    balance += amount if kind == "deposit" else -amount
print(balance)
# 120
print(sum(1 for kind, _ in events if kind == "deposit"))
# 2

orders = []
summary = {}


def place_order(customer, amount):
    orders.append((customer, amount))
    summary[customer] = summary.get(customer, 0) + amount


place_order("ana", 20)
place_order("ben", 5)
place_order("ana", 10)
print(summary)
# {'ana': 30, 'ben': 5}
```

Let's read it:

1. The events are only added to, never edited. Replaying them gives 100 - 30 + 50 = 120. The history also answers other questions, such as how many deposits there were: 2.
2. In the second part, `place_order` is a command. It appends to the log of orders, which is the write side.
3. The same command also updates `summary`, a read model shaped for one question: "how much has each customer spent?".
4. A query reads the summary directly: ana has spent 20 + 10 = 30, and ben 5. It does not need to scan every order.

If the summary were ever lost, it could be rebuilt by replaying `orders`. That is the safety net events provide.

## A common mistake {#mistake}
A common slip is using these patterns for a simple CRUD application, where they add much work and little value. Use them where history, auditing or very different read and write needs justify the extra parts.

A second slip is forgetting that the read side lags: a user who has placed an order may briefly not see it in their list.

## Key points {#key-points}
- CRUD is create, read, update and delete, and a single model is enough for simple rules.
- CQRS separates the model for changing data from the model for reading it.
- Event sourcing stores every change as an event, so the full history can be replayed.
- Both add complexity, so use them only where the benefits are needed.
