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
We can solve this by making sure that the class `Person` is only responsible for keeping the data and not responsible for formatting that data. Thus we will extract out the method `Format` and introduce another class that is responsible for formatting a book.

```
class PersonFormatter {
    public string Format(Person person, string formatType) {
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
```
class Person {
    public string FirstName { get; set; }
    public string LastName  { get; set; }
    public Gender Gender { get; set; }
    public DateTime DateOfBirth { get; set; }
}
```

We could argue that this is still a **violation**, as the `format` method itself still holds too much responsabilities as it has to implements too much details on how it is being formatted. We'll solve this later one with oneor more of the other principles: namely the **Dependency Inversion** principle and the **Interface Segragation Principle**.

## Open/Closed Principle
> Software entities (e.g. classes, modules, functions) should be open for extension but closed for modification.

### Violation example
### How to fix?

## Liskov Substitution Principle
> In a computer program, if type *B* is a subtype of type *A* then objects of type *A* may be replaced with objects of type *B* without altering any of the desirable properties of type *A*. In essence it means that every subclass should be substitutable for their base class. 

### Violation example
### How to fix?

## Interface Segregation Principle
> No client should be forced to depend on methods it does not use, implicitely implying that a client should never be forced to implement an interface it doesn't use.

### Violation example
### How to fix?

## Dependency Inversion Principle
> High-level modules should not depend on low-level modules, both should depend on abstractions. Abstractions should not depend on details, details should depend on abstractions.

### Violation example
### How to fix?