# Synchronous, asynchronous and message queues

**Covers:** Synchronous communication, Asynchronous communication, Message queue

## Waiting or not waiting {#overview}
When one part of a system needs another to do something, it has a basic choice: wait for the answer, or carry on and deal with the answer later. The choice affects speed, reliability and how tightly the two parts are tied together. In this article we explain both styles and the piece of software that makes the second one practical.

## Synchronous communication {#synchronous-communication}
Think of a phone call. You ask, and the other person answers right away. In synchronous communication, the caller sends a request and waits for the reply before it does anything else. Most web requests work this way.

It is easy to understand and to program, since the answer comes straight back. But it ties the two sides together in time. If the receiver is slow, the caller is slow. If it is down, the call fails. When a chain of services each wait for the next, one slow service at the end slows the whole chain, and the delays add up.

Synchronous calls suit cases where the caller truly needs the result before it can continue, such as checking whether a password is correct.

## Asynchronous communication {#asynchronous-communication}
Now think of an email. You send it and carry on with your day. In asynchronous communication, the caller sends a request and moves on immediately, without waiting. The answer, if any, arrives later or never.

The caller is no longer slowed by the receiver, and the receiver can be busy or briefly down without breaking the caller. But there are costs. The flow is harder to follow and to debug, you must decide what to do when no answer comes, and the user interface must handle a result that is not ready yet, such as "your order is being processed".

## Message queue {#message-queue}
If the caller moves on, where does the request wait? In a message queue, a durable middleman placed between a sender and a receiver. It holds messages so that the sender does not need the receiver to be online right now. The sender puts a message in the queue and carries on. The receiver takes messages out when it is ready, one at a time.

A queue adds several useful properties:
- **Decoupling in time:** sender and receiver need not run at the same moment.
- **Smoothing peaks:** if 1,000 orders arrive in one minute, they wait in line and the receiver works through them at its own pace.
- **Reliability:** messages are stored until they are handled, and if a receiver crashes, another can take over.
- **Scaling:** add more receivers to speed up processing.

Well-known tools include RabbitMQ, Apache Kafka and cloud services such as Amazon SQS. A group of tools like these is called a message broker.

## A worked example {#example}
A shop sends a confirmation email after each order.

```python
from queue import Queue

q = Queue()
for item in ("order-1", "order-2"):
    q.put(item)
print(q.qsize())
# 2
print(q.get())
# order-1
```

Let's read it:

1. The checkout code puts a message on the queue for each order and returns to the customer at once. It does not wait for any email to be sent.
2. Two messages are waiting, so the queue size is 2.
3. The email worker takes the oldest message first: `order-1`.

Now compare the two styles with numbers. Suppose sending an email takes 2 seconds and 300 orders arrive in one minute.
- **Synchronous:** each customer waits 2 seconds at checkout. If the email service is down, checkouts fail.
- **Asynchronous with a queue:** checkout takes a few milliseconds. The queue holds 300 messages, and one worker needs 300 x 2 = 600 seconds, which is 10 minutes, to clear them. Adding 5 workers cuts that to 600 divided by 5 = 120 seconds, 2 minutes. If the email service is down for 5 minutes, nothing is lost.

The customer sees a fast checkout, and the system absorbs the peak.

## A common mistake {#mistake}
A common slip is using a queue for something the caller must know immediately, such as "is this card valid?". Use synchronous calls when the answer is needed now.

A second slip is forgetting that messages can be delivered twice. Design the receiver so that handling the same message again does no harm.

## Key points {#key-points}
- Synchronous communication waits for the reply. It is easy to follow but ties the two sides together.
- Asynchronous communication moves on without waiting, which gives resilience and speed with more complexity.
- A message queue holds messages between sender and receiver, smoothing peaks and surviving outages.
- Make receivers safe against duplicate messages.
