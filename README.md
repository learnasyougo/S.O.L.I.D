# S.O.L.I.D: The First 5 Principles of Object Oriented Design.
 - [Single Responsibility Principle](#single-responsibility-principle)
 - [Open/Closed Principle](#openclosed-principle)
 - [Liskov Substitution Principle](#liskov-substitution-principle)
 - [Interface Segregation Principle](#interface-segragation-principle)
 - [Dependency Inversion Principle](#dependency-inversion-principle)

## Single Responsibility Principle
> A class should only have only one reason to change.

### Violation example
Class `Person` is responsible for holding person related data, it also holds has a function `Format` which outputs this person data to a given format. Currently this method accepts a parameter on which the function decides which algorithm to use return the data in the required format. 

The example below **violates** the **single responsibility principle** because the class `Person` now is responsible for keeping data of of the person **and** formatting that data. 
```
class Person {
    public string FirstName { get; set; }
    public string LastName  { get; set; }
    public Gender Gender { get; set; }
    public DateTime DateOfBirth { get; set; }
    public string Format(string formatType) {
        switch(formatType) {
            case "JSON":
               // implement JSON formatting here
               return jsonFormattedString;
               break;
            case "FirstAndLastName":
              // implementation of first & lastname formatting here
              return firstAndLastNameString;
              break;
            default:
              // implementation of default formatting
              return defaultFormattedString;
        }
    }
}
```
### How to fix?
We can solve this by making sure that the class `Person` is only responsible for keeping the data and not responsible for formatting that data itself. Thus we will extract out the details of the method `Format` and introduce another class that is responsible for formatting a person.

```
interface IFormatter {
    string Format(Object @object);
}
```
```
class JSONFormatter : IFormatter {
    public string Format(Object @object) {
        // implement collecting object's public properties & values and formatting
        // it in JSON afterwards
        return jsonFormattedString;
    }
}
```
```
class Person {
    public string FirstName { get; set; }
    public string LastName  { get; set; }
    public Gender Gender { get; set; }
    public DateTime DateOfBirth { get; set; }
    public string Format(IFormatter formatter) {
        formatter.Format(this);
    }
}
```

We could argue that this example still holds **violations** against **SOLID**, and that is true. But at least now we can change the implementation of the formatting of a Person without affecting the Person class itself - as the SRP states.

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