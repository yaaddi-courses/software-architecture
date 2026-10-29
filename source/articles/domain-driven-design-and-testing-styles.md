# Domain-driven design, TDD and BDD

**Covers:** Domain-driven design, DDD, Bounded context, TDD, BDD

## Building what the business means {#overview}
Many software failures are failures of language. Developers build what they understood, and the business meant something else. One family of ideas tackles this by shaping the code around the real domain and its vocabulary. Two testing styles, which start from the expected behavior, fit well with it. In this article we cover the design approach first and the two ways of testing second.

## Domain-driven design {#domain-driven-design}
The domain is the area of knowledge the software serves, such as shipping, banking or insurance. Domain-driven design models the software around that real business domain, keeping the vocabulary of the code close to the language of the people who work in that field. Eric Evans described the approach in a book published in 2003.

Its core habit is a shared language. Developers and domain experts agree on terms, such as "policy", "claim" and "premium", and use exactly those words in conversation, in documents and in code. If a claims handler says "settle a claim", the code has a method called `settle_claim`, not `process_item`. When the words match, misunderstandings show up early.

## DDD {#ddd}
DDD is the standard short form of domain-driven design. In meetings you might hear "this is a DDD question", meaning that the answer depends on how the business sees the concept.

## Bounded context {#bounded-context}
Here is a problem the shared language runs into: the same word can mean different things in different parts of a company. To the sales team, an "order" is a customer's request with prices and discounts. To the warehouse, it is a list of items to pick and pack. To accounting, it is something to invoice.

A bounded context is a clear boundary around the area in which one particular model and its terms apply. Trying to build one universal "Order" for all three produces a bloated, confusing model. Domain-driven design instead draws a boundary around each meaning, with its own model, and defines how the contexts exchange information. A bounded context is often a good candidate for a module, or for a service of its own.

## TDD {#tdd}
TDD, or test-driven development, is a way of working in short loops: write a small failing test, write only enough code to pass it, and then clean up. The test describes the behavior you want before the code exists, which keeps the design focused on use.

## BDD {#bdd}
BDD, or behavior-driven development, describes software by examples of its behavior in plain language that business people can read and check. A common format is Given, When, Then: "Given a customer with a gold card, when they spend 100 dollars, then they earn 20 points."

BDD grew out of TDD and brings in the shared language of domain-driven design. The examples become automated tests, so the specification is always checked against the real software.

## A worked example {#example}
A bank has two bounded contexts for the word "account". Let's compare them.

| Context | What "account" means | What it holds |
|---|---|---|
| Banking | A place where money is held | Balance, owner, overdraft limit |
| Marketing | A customer's profile for campaigns | Email, interests, consent |

The two contexts are separate models. They share only the customer's id, and each can change without breaking the other. A message such as "customer 42 opened a savings account" travels between them.

Now a BDD example for the banking context, in the shared language:

```text
Given an account with a balance of 50 dollars
When the owner withdraws 80 dollars
Then the withdrawal is refused
And the balance is still 50 dollars
```

A developer turns this into a test, using TDD: first the test fails, because the rule does not exist. Then the rule is written and the test passes. The business expert can read the example and say, "the overdraft limit is 100, so this should be allowed". That one correction, found in a five-line example, saves a whole release.

## A common mistake {#mistake}
A common slip is using domain-driven design for a simple system that is mostly storing and showing data. It adds effort without payoff. It shines where the business rules are complex.

A second slip is writing BDD examples that are really long technical scripts, which the business cannot read.

## Key points {#key-points}
- Domain-driven design shapes code around the real business and its shared language.
- A bounded context is a boundary where one model and its words have one meaning.
- TDD writes a failing test first. BDD writes examples in plain Given, When, Then language.
- Use these for complex domains, not for simple data entry.
