# 3 Tier Architecture

In some traditional web project, we often find the some common package name like `dao`, `service`, `controler` and `model`.
In these project, it group all the code into 3 packages(tiers) - `Presentation`, `Business logic` and `Data access`.
Actually, we call these project are built with 3 tiers architecture.

## What's 3 Tier Architecture

> David Wheeler: All problems in computer science can be solved by another level of indirection

A system can be divided into 3 tier from upstream to downstream, the different tier must have strict dependency
which is upper tier depends on lower tier, and lower tier should not depends on upper tier.

The explicit boundary can make the system decouple with each tier. The responsibility of each tier will be much more explicit.

Meanwhile, the lower tier is much more stable than upper tier.

## DIP and 3 tier

DIP is short for Dependency inversion principle, one important principle of SOLID principle.
DIP says
> High level modules should not depend upon low level modules. Both should depend upon abstractions.
> Abstractions should not depend upon details. Details should depend upon abstractions.

We know that in some traditional java project, we always make the code dependency like this - `controller => service => dao`.
So we always define the interface between different tier, the dependency should be like
`controller => interface <= service => interface <= dao `


So this is the [Hexagonal architecture](http://alistair.cockburn.us/Hexagonal+architecture) style which is called port and adapter architecture.
Port is used to handle output and input data, every port should have an adapter.

Actually, 3 tier architecture don't have the guidance about how to organize the code such as email sending and queue operation.

In DDD, domain logic is the most important assets, in DDD we can use this architecture to make the domain logic to decouple with the input/output,
and defer to implement the code which is out of domain.
