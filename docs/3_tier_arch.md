# What's Architecture

When the some people talked about `Architecture`, they treat it as an very high level decision, which exclude all details.
Actually, an software architecture include all lower level design details and include top level arch info.
They can't be divided to 2 part, given that all the detail is the pillar for the high level architecture.

Refer to the [C4 model](https://c4model.com/), it's an useful tool to describe the software architecture.
Follow the C4 model, we can use `Context, Containers, Components, and Code` - 4 level to describe an arch.

## 2 dimensions of an architecture value

### Behavior value

The work of a developer is making the machine works to meet the business requirements, so the customer/company can benefit from the program.
When the machine have unexpected behavior, developer should to debug and solve it.

In shortly, we can say make code work. Some people will say it's all the work for the developer, is it? NO!

### Architecture value

When we say `software` vs `hardware`, the `soft` indicate the Flexibility which means we can easily change the behavior of an machine with the `software`.

For that goal, so the software should be much more soft, in other words, it should be easily be changed for the different business requirements.

The cost for every change should fixed. But in some system, the code for change will be higher and higher with the growth of the feature.

### Which dimension is important?

All the thing can be group by important or urgent.

One is urgent but not always important.
One is important but not always urgent.

| important and urgent      | important but not urgent     |
|---------------------------|------------------------------|
| not important, but urgent | not important and not urgent |

Ideally, people should focus on the thing is important. But in real world, we always treat the urgent thing as the high priority.

The important thing is that if people ignore the architecture value of the software, the system will be hard to maintain.

# How to build an architecture

Like the high quality bricks to a building, a good software system should start from the clean code. We can follow the practice in the book `Clean Code`.
But if the arch design of the building is not good, even we have good quality bricks, the building is not good as it can be.

So we can introduce the design principle to solve this problem. We can refer to the book - `Agile Software Development - Principles, Patterns, and Practices`

## Design principle - SOLID

Why we need to design principle, in order to
1. Making software can be tolerant of change.
2. Making software can be understand easily.
3. Make the component can be reuse in multi software system.

### SRP - Single Responsibility Principle

Some developer will say this means `Every module should on only one thing`, such as one function only have one functionality.
But this is a principle facing low level detail implementation.

That's is not all, `Every software module only have one reason for change`.

`module` means a couple of function and data structure related with each other closely.

### OCP - Open/Close

A good software should be open to extend and Close to modify.

The typeclass is a good example for the principle.
```
val p = Man(name, gender)
def printMan(man: Man, showMan: Show[Man]): String = man.show
```
The `printMan` is open to extend, if we want to show man in different way, just pass the different `Show` instance for man.

### LSP

A principle for how to inherit, refer to https://stackoverflow.com/questions/56860/what-is-an-example-of-the-liskov-substitution-principle

### ISP - Interface segregation

No client should be forced to depend on methods it does not use.

```
Class Ops {
 def ops1
 def ops2
 def ops3
}
class User1 { ops.ops1 }
class User2 { ops.ops2 }
class User3 { ops.ops3 }
```

According to the ISP, ideally, `user1` should depend on the `ops1` interface, given we don't want make the `User1` to recompile when change `Ops` implementation.
This principle can guide us to build a better library.

Following the typeclass example, the `printPeople` function can be never re-compiled, whatever the `Man` is.

```
val p = Man(name, gender)
def printPeople[T](man: T, show: Show[T]): String = man.show
```

### DIP - dependency inversion

1. High-level modules should not depend on low-level modules. Both should depend on abstractions (e.g., interfaces/algebra).
2. Abstractions should not depend on details. Details (concrete implementations) should depend on abstractions.

```
class UserService(dao: UserDao) {
  def saveUser(): Unit = dao.saveUser
}

trait UserDao { def saveUser }

class PostgresUserDao extend UserDao {...}
class MemUserDao extend UserDao {...}

```

It seems that the `UserService` is depends on `UserDao`. But actually, `UserDao` is belongs to `UserService`, they are in same level abstractions/layer.
So `PostgresUserDao` and `MemUserDao` is depends on the `UserDao`. That is one of the inversion.

## The goal of design architecture

The system architecture is support for the use case.
The arch design should be irrelevant with the framework, tools and running environment.

## 3 layer Architecture

In some traditional web project, we often find the some common package name like `dao`, `service`, `controler` and `model`.
In these project, it group all the code into 3 packages(layers) - `Presentation`, `Business logic` and `Data access`.
Actually, we call these project are built with 3 layers architecture.

Here is a document introduce layer architecture: https://herbertograca.com/2017/08/03/layered-architecture/

> David Wheeler: All problems in computer science can be solved by another level of indirection

A system can be divided into 3 layer from upstream to downstream, the different layer must have strict dependency
which is upper layer depends on lower layer, and lower layer should not depends on upper layer.

The explicit boundary can make the system decouple with each layer. The responsibility of each layer will be much more explicit.

Meanwhile, the lower layer is much more stable than upper layer.

The advantages are:

* We only need to understand the layers beneath the one we are working on;
* Each layer is replaceable by an equivalent implementation, with no impact on the other layers;
* Layers are optimal candidates for standardisation;
* A layer can be used by several different higher-level layers.

The disadvantages are:

* Layers can not encapsulate everything (a field that is added to the UI, most likely also needs to be added to the DB);
* Extra layers can harm performance, especially if in different tiers.

## Ports & Adapters Architecture

User cases/business logic is the core of a system, if we want to isolate our application use case, we can do it using entry/exit points.

So this is the [Hexagonal architecture](https://herbertograca.com/2017/09/14/ports-adapters-architecture/) style which is called port and adapter architecture.

* A port is a consumer agnostic entry and exit point to/from the application.
* An adapter is a class that transforms (adapts) an interface into another.

Port is used to handle output and input data, it can be a interface/algebra
Every port should have at least one adapter, which means adapt to the `interface/algebra`.

We know that in some traditional java project, we always make the code dependency like this - `controller => service => dao`.
So we always define the interface between different layer, the dependency should be like
`controller => interface <= service => interface <= dao `.

Actually, 3 layer architecture don't have the guidance about how to organize the code such as email sending and queue operation which is 3rd resource access.
In general, these code will be put into the service layer.

### What are the benefits?

> Using this port/adapter design, with our application in the centre of the system, allows us to keep the application isolated from the implementation details like ephemeral technologies, tools and delivery mechanisms, making it easier and faster to test and to create a reusable proof of concept.
