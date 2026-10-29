# Coupling, cohesion, separation of concerns and SOLID

**Covers:** Coupling, Cohesion, Separation of concerns, SOLID

## The ideas behind good structure {#overview}
Architecture styles come and go, but a few principles hold in all of them. They answer one question: how do you arrange code so that a change in one place does not break ten others? In this article we will cover four related ideas, from the plainest measure of tangling to a set of five guidelines for object-oriented design.

## Coupling {#coupling}
Coupling describes how tangled two parts of a system are with each other. With high coupling, a change in one part forces changes in the other. With low coupling, one part can change without the other noticing.

Signs of high coupling are parts that share the same data tables, that know each other's internal details or that must be released together. Low coupling comes from small, stable interfaces: a part says what it offers and hides how. The aim is low coupling between parts.

## Cohesion {#cohesion}
Now look inside one part. Cohesion describes how focused it is on one clear job. A module with high cohesion does one thing, and everything in it belongs together. A module named "Utilities" that contains date handling, email sending and tax calculation has low cohesion, and nobody can tell what it is for.

The aim is the pair: high cohesion inside each part and low coupling between parts. Together they make code easy to understand and to change.

## Separation of concerns {#separation-of-concerns}
How do you get that pair in practice? Keep each distinct responsibility in its own independent part, and do not tangle unrelated things together. That principle is separation of concerns. Showing a page, applying a discount rule and saving to a database are different concerns. If one function does all three, a change to the page layout risks breaking the discount.

Separation of concerns is the idea that produces both layers and modules. It is the means, and low coupling with high cohesion is the result.

## SOLID {#solid}
SOLID is a set of five guidelines for object-oriented design, collected by Robert C. Martin. The letters stand for:

- **S, single responsibility:** a class should have one reason to change.
- **O, open/closed:** open for extension, closed for modification. You add behavior without editing working code.
- **L, Liskov substitution:** a subtype must be usable wherever its parent type is expected, without surprises.
- **I, interface segregation:** prefer several small interfaces over one large one, so no one depends on what they do not use.
- **D, dependency inversion:** depend on abstractions, not on concrete details. High-level code should not know which database or sender is used.

They are guidelines, not laws. Applied without judgment, they can produce too many tiny classes.

## A worked example {#example}
The following program separates the "what to send" from the "how to send it", which reduces coupling and follows dependency inversion.

```python
class SmsSender:
    def send(self, message):
        return f"SMS: {message}"


class EmailSender:
    def send(self, message):
        return f"EMAIL: {message}"


def notify(message, sender):
    return sender.send(message)


print(notify("hi", SmsSender()))
print(notify("hi", EmailSender()))
# SMS: hi
# EMAIL: hi
```

Let's read it:

1. `notify` has one job: pass a message to a sender. That is cohesion.
2. It does not know which sender it receives, as long as it has a `send` method. That is low coupling and dependency inversion.
3. To add a push notification, you write a new class and change nothing in `notify`. That is open/closed.
4. Both senders can be used wherever a sender is expected. That is Liskov substitution.
5. The concern of deciding what to say is separate from the concern of delivering it. That is separation of concerns.

Now count the cost of a change. If `notify` had built-in checks such as `if kind == "sms"`, adding a third channel would mean editing it, and each edit risks breaking the first two. With the design above, a new channel adds one class and zero edits to working code.

## A common mistake {#mistake}
A common slip is applying these ideas mechanically: a single-method class for every tiny task, or an interface with only one implementation and no plan for another. Add structure where change is likely, and keep plain code plain.

A second slip is confusing the two measures. You want high cohesion and low coupling, not the reverse.

## Key points {#key-points}
- Coupling is how much parts depend on each other. Aim for low.
- Cohesion is how focused a part is on one job. Aim for high.
- Separation of concerns keeps different responsibilities in different places.
- SOLID is five guidelines for object-oriented design. Use judgment.
