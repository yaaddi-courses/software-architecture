# Quality attributes and recording decisions

**Covers:** Quality attribute, ADR, RFC, SLA

## What architecture is for {#overview}
A system has two kinds of requirements. The first kind is features: what the software does. The second kind is how well it must do it: how fast, how reliable, how secure, how easy to change. Architecture is mostly about the second kind. The big structural choices are the ones that make a system fast or slow, robust or fragile, cheap or expensive to change. Because they are hard to reverse, they deserve a clear goal and a written record.

## Quality attribute {#quality-attribute}
A feature says "users can pay by card". A quality attribute is a goal that goes beyond features, such as scalability, reliability, security, performance or maintainability. They are sometimes called the "-ilities". For the same payment, a quality attribute says "payment completes in under two seconds, even on the busiest day".

Quality attributes often pull against each other. Making a system very secure may slow it. Making it very flexible may make it harder to understand. Architecture means choosing which attributes matter most for this system and accepting a cost in others. A bank may put security and correctness first. A game may put speed first.

To be useful, a quality attribute needs a number. "The system should be fast" cannot be checked. "95 percent of searches return within 300 milliseconds with 1,000 users at once" can.

## SLA {#sla}
An SLA, or service-level agreement, is a promise about how reliable or fast a service will be, often written into a contract with customers. A typical SLA states availability, such as 99.9 percent uptime a month, and what happens if it is missed, such as a refund.

An SLA turns a quality attribute into a commitment. The architecture must be able to meet it, so compute what it allows. A 30-day month has 43,200 minutes. At 99.9 percent availability, the allowed downtime is 0.1 percent of that, about 43 minutes. At 99.99 percent, it is 0.01 percent, only about 4 minutes.

## RFC {#rfc}
An RFC, or request for comments, is a written proposal that others review before a decision is made. The name comes from the early internet, where standards were published as numbered "requests for comments". Many teams use the same idea for design changes: an author writes the problem and the proposed solution, colleagues comment and the author revises.

An RFC invites disagreement while it is cheap. It also gives people who are affected, but not in the room, a chance to speak.

## ADR {#adr}
Once the decision is made, how do you make sure the reasons survive? An ADR, or architectural decision record, is a short written note that captures an important decision and the reason for it. Months later, a new team member asks "why did we pick this database?" and the answer is in the file, not lost with the person who left.

A good record is short, often one page, and has a few standard parts: the context, the decision, the alternatives considered and the consequences, good and bad. Records are kept in the project's repository and never rewritten. If a decision changes, a new record replaces it and refers to the old one.

An RFC is the discussion before a decision, and an ADR is the outcome afterwards.

## A worked example {#example}
A team must choose how to store product images. This is the ADR they write:

```markdown
# ADR 7: Store product images in object storage

Status: accepted (replaces nothing)

Context: Product pages must load in under 2 seconds. Images are 4 MB on
average, and we expect 50,000 images in year one.

Decision: Store images in a cloud object storage service, not in the database.

Alternatives considered: database blobs (slow backups), shared disk
(does not scale).

Consequences: Good: cheap and fast to serve. Bad: a new service to
operate, and a second place to back up.
```

The matching quality attributes are performance (under 2 seconds) and cost. The SLA the shop has promised customers is 99.9 percent availability. Check the budget: 99.9 percent of a 30-day month (43,200 minutes) leaves 0.1 percent for downtime, which is 43.2 minutes. The team now knows that a single outage of an hour would break the promise, which supports its choice of a managed storage service.

Before the ADR, the team shared the idea in an RFC for a week. Two engineers raised the backup problem, which is why it appears in the consequences.

## A common mistake {#mistake}
A common slip is writing quality goals without numbers, or recording decisions only in chat. In both cases the reasoning disappears.

A second slip is writing an ADR after the work is finished, from memory. Write it while the alternatives are fresh.

## Key points {#key-points}
- Quality attributes are goals beyond features, and need numbers to be useful.
- An SLA turns a quality goal into a promise. Calculate the downtime it allows.
- An RFC invites review before a decision, and an ADR records the decision and its reasons.
- Keep records short and never rewrite them.
