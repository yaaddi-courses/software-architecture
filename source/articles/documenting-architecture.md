# Documenting architecture: the C4 model, UML and ERDs

**Covers:** C4 model, C4, UML, ERD

## Pictures that people can actually read {#overview}
An architecture exists in people's heads and in the code, and neither is easy to share. A diagram helps, but diagrams often fail. Either one huge picture holds every box and arrow and confuses everyone, or the picture uses symbols nobody remembers. A good approach is to draw several plain pictures at different zoom levels, each for a different audience.

## C4 model {#c4-model}
The C4 model documents architecture at four zoom levels, each going further in from the widest view. The four Cs are:

1. **Context:** the system as a single box, surrounded by the people and other systems that use it. This is for everyone, including non-technical people.
2. **Containers:** the major runnable pieces inside the system, such as a web app, an API, a database and a message queue, with how they talk. A "container" here means any separately running part, not the Docker kind.
3. **Components:** the main building blocks inside one container, such as an order controller or a payment module.
4. **Code:** the classes and functions inside a component. This level is rarely drawn by hand, since tools can generate it.

Simon Brown created the model. Most teams draw the first two levels, and the third only for the important containers. The method is easy: boxes with a name, a short description of the role and labelled arrows saying what flows between them, with a small legend.

## C4 {#c4}
"C4" is the usual shorthand: "put it in a C4 diagram" means a picture at one of those levels. It is not a drawing tool or a standard of notation, so you can draw it on a whiteboard or in any tool.

## UML {#uml}
UML, or Unified Modeling Language, is a standard set of diagram types for describing software designs. It includes class diagrams, which show classes and their relationships, sequence diagrams, which show the order of messages between parts over time, and use case diagrams, which show how users interact.

UML is formal, and it was once used for very detailed designs. Today most teams use only a few kinds, often sketched informally. A sequence diagram is the most widely useful, since it shows who calls whom and in what order for one scenario. The risk is over-detail: a UML picture that nobody keeps up to date is worse than none.

## ERD {#erd}
An ERD, or entity-relationship diagram, is a picture of the tables in a database and how they relate. Each box is an entity, usually a table, with its columns, and lines between boxes show relationships such as "one customer has many orders".

The symbols at the ends of a line tell you the count: one, many, zero-or-one. An ERD is the standard way to discuss a data design before it is built and to explain an existing database to a new person.

## A worked example {#example}
A small shop documents its system with C4 and an ERD. Let's start with C4.

| Level | What the picture shows |
|---|---|
| Context | The "Shop" system in the center. Customers use it through a browser. It uses an external payment provider and sends emails through an email service |
| Containers | Web app, order API, database, message queue. The web app calls the API. The API stores data in the database and puts a "send receipt" message on the queue |
| Components | Inside the order API: order controller, payment module, receipt sender |

For the data, here is the ERD in text form:

```text
CUSTOMER (id, name, email)  1 ---- * ORDER (id, customer_id, created_at)
ORDER    (id, ...)          1 ---- * ORDER_LINE (order_id, product_id, quantity)
PRODUCT  (id, name, price)  1 ---- * ORDER_LINE
```

Read it like this: one customer has many orders, and one order has many order lines. A product can appear in many order lines. The ORDER_LINE table connects orders and products, which is how a many-to-many relationship is stored.

Count the arrows to check the picture. The Containers level has 4 boxes. If each talks to at most 2 others, that is no more than 4 arrows (4 boxes x 2 links, counted once from each end), which is small enough to read at a glance. A single diagram showing all 40 classes would need hundreds of arrows and help nobody.

## A common mistake {#mistake}
A common slip is mixing levels in one picture, such as a database table next to a whole external system. Keep each diagram at one zoom level and say what it is.

A second slip is drawing once and never updating. Keep the diagrams in the repository next to the code, and update them when the design changes.

## Key points {#key-points}
- The C4 model has four zoom levels: context, containers, components and code.
- Draw the first two levels for most systems. Keep each picture at one level.
- UML is a standard set of diagrams. A sequence diagram is often the most useful.
- An ERD shows the tables of a database and their relationships.
