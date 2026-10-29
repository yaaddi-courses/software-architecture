# REST, HTTP and API gateways

**Covers:** REST, API, HTTP, POST, API gateway

## How programs ask each other for things {#overview}
Parts of a modern system rarely share memory. They send requests to each other over a network, following agreed rules. Learning those rules is the key to reading almost any web-based design. In this article we explain the vocabulary of these requests, a popular style for organizing them, and a single front door that protects many services behind it.

## API {#api}
An API, or application programming interface, is a defined way for one program to ask another for something. It describes what you can ask, how you must ask and what comes back, without showing how the other program works inside. Think of a restaurant menu: you choose from the list and the kitchen's methods stay hidden.

Good APIs are stable. Many other programs depend on them, so changing them carelessly breaks those programs.

## HTTP {#http}
Most web APIs speak HTTP, or HyperText Transfer Protocol, the protocol that web browsers and servers use to exchange pages and data. A request has a method that says what you want to do and a path that says what to do it to, such as `GET /orders/42`. The response has a status code and usually a body.

The common methods are GET to read, POST to create, PUT or PATCH to change and DELETE to remove. The status codes you will see most are 200 (success), 201 (created), 400 (the request was wrong), 401 or 403 (not allowed), 404 (not found) and 500 (the server failed).

## POST {#post}
POST is the method a client uses to send data to a server, normally to make a new record. Submitting a signup form sends a POST with the data in the request body. Unlike GET, it is not safe to repeat blindly. Sending the same POST twice may create two orders, which is why systems add protections such as unique request keys.

## REST {#rest}
REST is an API design style that organizes everything around resources, things with names such as orders and customers, and uses the HTTP methods for the actions on them. Each request carries everything the server needs, and each response has a status code.

A REST API for orders has a few predictable addresses:

| Request | Meaning |
|---|---|
| `GET /orders` | List orders |
| `GET /orders/42` | Read order 42 |
| `POST /orders` | Create an order |
| `PUT /orders/42` | Replace order 42 |
| `DELETE /orders/42` | Delete order 42 |

Use nouns in the paths, not verbs: `/orders`, not `/createOrder`. The predictability is the benefit, because a developer who knows the pattern can guess most of the API.

## API gateway {#api-gateway}
When a system has many services, clients should not need to know them all. An API gateway is a single front door that routes incoming requests to many backend services, and often protects them. Clients talk to one address, and the gateway forwards `/orders` to the order service and `/users` to the user service.

Because every request passes through it, the gateway is the natural place for shared jobs: checking who the caller is, limiting the request rate, logging, caching and turning one outside request into several inside ones. The cost is that it is an extra step on every request, and a single point of failure unless it is run in several copies.

## A worked example {#example}
A shop has three services behind a gateway. Let's follow `POST /orders` with a new order.

1. The app sends `POST /orders` to the gateway with the order in the body.
2. The gateway checks the login token. Without a valid one it answers 401 and goes no further.
3. It checks the rate limit for this caller, then routes the request to the order service.
4. The order service creates the order and replies 201 (created) with the new order's address, `/orders/43`.
5. The gateway passes the response back to the app.

If the app then sends `GET /orders/999` and no such order exists, the service answers 404.

Now count the benefit. With 3 services and no gateway, each service must implement login checks and rate limits, 3 copies of the same code. With a gateway, there is 1 copy. The cost is a few milliseconds of extra time per request, which for most systems is a good trade.

## A common mistake {#mistake}
A common slip is putting business rules in the gateway. It then becomes a hidden second application that everyone depends on. Keep it to routing and shared checks.

A second slip is treating every action as a POST to one address. That throws away the predictability that REST was meant to give.

## Key points {#key-points}
- An API is the agreed way for programs to ask each other for things.
- HTTP requests have a method, a path and a status code. POST sends data to create something.
- REST organizes an API around resources with predictable paths and the standard methods.
- An API gateway is one front door that routes and protects requests to many services.
