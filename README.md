# S.O.L.I.D: The First 5 Principles of Object Oriented Design.
 - [Single Responsibility Principle](#single-responsibility-principle)
 - [Open/Closed Principle](#openclosed-principle)
 - [Liskov Substitution Principle](#liskov-substitution-principle)
 - [Interface Segregation Principle](#interface-segragation-principle)
 - [Dependency Inversion Principle](#dependency-inversion-principle)

## Single Responsibility Principle
> A class should only have only one reason to change.
- It relates strongly to cohesion and coupling: strive for high *cohesion*, but for low *coupling*.
- More responsibilities in a module makes it more likely to change.
- The more modules a change affects, the more likely the change will introduce an error.
- Therefore, *a class should only have one reason to change* and *correspond to have one single responsibility*.

*Cohesion*: how strongly-related and focused are the various elements of a module (class, function, namespace...)
*Coupling*: the degree to which each module relies on each on of the other modules.

### Violation example

### How to fix?

#### Resources & Links
- https://code.tutsplus.com/tutorials/solid-part-1-the-single-responsibility-principle--net-36074
- http://www.oodesign.com/single-responsibility-principle.html
- http://softwareengineering.stackexchange.com/questions/342051/violation-solution-for-single-responsibility-principle

## Open/Closed Principle
> Software entities (e.g. classes, modules, functions) should be open for extension but closed for modification.

### Violation example
Building on from the `PersonFormatter` class we introduced in the *how to fix* of the [Single Responsibility Principle](#how-to-fix) we see that if we'd want to introduce a a new format, we have to change the `PersonFormatter` class each time, as it will probably rely on different other classes to help serialize to either *json, xml, ...*. This clearly violates the *open/close*s

### How to fix?

#### Resources & Links
- http://www.oodesign.com/open-close-principle.html

## Liskov Substitution Principle
> In a computer program, if type *B* is a subtype of type *A* then objects of type *A* may be replaced with objects of type *B* without altering any of the desirable properties of type *A*. In essence it means that every subclass should be substitutable for their base class. 

### Violation example
### How to fix?

#### Resources & Links
- http://www.oodesign.com/liskov-s-substitution-principle.html

## Interface Segregation Principle
> No client should be forced to depend on methods it does not use, implicitely implying that a client should never be forced to implement an interface it doesn't use.

### Violation example
### How to fix?

#### Resources & Links
- http://www.oodesign.com/interface-segregation-principle.html

## Dependency Inversion Principle
> High-level modules should not depend on low-level modules, both should depend on abstractions. Abstractions should not depend on details, details should depend on abstractions.

### Violation example
### How to fix?

#### Resources & Links
- http://www.oodesign.com/dependency-inversion-principle.html