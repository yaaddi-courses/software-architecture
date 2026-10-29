# Monoliths, layers and clean boundaries

**Covers:** Monolith, DAL, Modular monolith, Hexagonal architecture, Clean Architecture, UI, UIs

## One program, several ways to organize it {#overview}
The simplest way to build a system is to put it all in one program. That can go well or badly, depending on how the inside is organized. This article follows a path from a plain single program, through layers and strict module boundaries, to two styles that protect the heart of the application from the outside world.

## Monolith {#monolith}
A monolith is an application that is built, deployed and run as one single unit. All features live in the same codebase and are released together. It is simple to start with: one thing to build, one thing to run, easy to debug, and calls between parts are fast function calls.

The problems appear as it grows. Many people edit the same code and collide. A small change needs the whole system to be tested and released. A weak part can drag down the rest. Still, a monolith is often the right first choice. Many successful products ran as monoliths for years, and splitting too early into many services adds more complexity than it removes.

## UI {#ui}
UI, or user interface, is the part of the software that people see and use: screens, buttons and forms. In architecture, the UI is one of the outer edges of the system. It changes often, as fashions and devices change, so a good design keeps important business rules out of it.

## DAL {#dal}
A DAL, or data access layer, is the part of a program that talks to the database. It hides the details of queries and storage behind simple functions such as "find customer by id". The rest of the code does not need to know whether the data lives in one database or another.

This is the idea of layering: group code by role, and let each group talk only to the one beneath it. A typical stack is a UI layer, a business rules layer and a data access layer. Layers make code easier to find and to replace one piece at a time.

## Modular monolith {#modular-monolith}
A modular monolith is a single deployable application with strict boundaries between its internal modules. Each module, such as orders, payments or accounts, has a small public surface, and other modules may use only that surface, never reach into its inner workings.

It keeps the simplicity of one deployment and gains much of the order of separate services. Tools can check the boundaries automatically and fail the build when a module reaches across. If a module later needs to become its own service, the clear boundary makes that split far easier.

## Hexagonal architecture {#hexagonal-architecture}
Hexagonal architecture, also called ports and adapters, isolates the core logic of a system from the outside world. The core defines ports, which are small interfaces describing what it needs or offers: "save an order", "send a notification". Adapters connect the ports to real things: a web screen, a database, an email service.

The benefit is that the core can be tested without a database or a screen, and technologies can be swapped by writing a new adapter. The hexagon shape in the diagrams has no special meaning. It only shows that there are many sides for adapters.

## Clean Architecture {#clean-architecture}
Clean Architecture is a style, popularized by Robert C. Martin, built as concentric circles with the business rules at the center. The key rule is that dependencies point inward. The inner circles know nothing about the outer ones. The database, the UI and the frameworks sit on the outside and can be replaced without touching the rules.

It is close in spirit to hexagonal architecture. Both aim to keep what matters most, the business rules, free from technical details that change.

## A worked example {#example}
An online shop starts as one program. The team wants to make it easier to maintain without splitting it up.

| Step | Change | Effect |
|---|---|---|
| 1 | A monolith: one codebase and one deployment | Simple, but tangled |
| 2 | Add layers: UI, rules, DAL | Code is easier to find; the database can be swapped inside the DAL |
| 3 | Make it a modular monolith: orders, payments and accounts modules with public surfaces | Modules cannot reach into each other |
| 4 | Apply ports and adapters: the order rules define a "save order" port | Tests run in 2 seconds without a database |

Check the result with arithmetic: before step 4, the 400 tests needed a database and took 8 minutes. After it, 360 of them use an in-memory adapter and take about 3 seconds in all. Only 40 tests still use the real database, and they take about 1 minute. The run time falls from 8 minutes to about 1 minute, which makes developers run the tests much more often.

The deployment stayed a single unit throughout, which is why the monolith label still applies.

## A common mistake {#mistake}
Believing that a monolith is bad and that splitting into services is always progress. Many teams swap one tangled program for many tangled services. Fix the boundaries first. A second mistake is building many layers and ports before there is any need, adding code that nobody uses.

## Key points {#key-points}
- A monolith is deployed as one unit. It is simple to start and gets harder to change as it grows.
- Layers, such as a UI, rules and a DAL, group code by role.
- A modular monolith adds strict module boundaries inside one deployment.
- Hexagonal and Clean Architecture keep the business rules free from the database, the UI and frameworks.
