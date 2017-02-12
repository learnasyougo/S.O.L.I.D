# S.O.L.I.D: The First 5 Principles of Object Oriented Design.
 - [Single Responsibility Principle](#single-responsibility-principle)
 - [Open/Closed Principle](#openclosed-principle)
 - [Liskov Substitution Principle](#liskov-substitution-principle)
 - [Interface Segregation Principle](#interface-segragation-principle)
 - [Dependency Inversion Principle](#dependency-inversion-principle)

 *Note: The violation & solution examples given build from top to bottom, they improve the code and design step by step/principle by principle.*

## Single Responsibility Principle
> A class should only have only one reason to change.
- It relates strongly to cohesion and coupling: strive for high *cohesion*, but for low *coupling*.
- More responsibilities in a module makes it more likely to change.
- The more modules a change affects, the more likely the change will introduce an error.
- Therefore, *a class should only have one reason to change* and *correspond to have one single responsibility*.

*Cohesion*: how strongly-related and focused are the various elements of a module (class, function, namespace...)<br/>
*Coupling*: the degree to which each module relies on each on of the other modules.

### Violation example
In this demonstration we have a Person class holding a `FirstName`, `LastName`, `DateOfBirth` and `Gender`. There's also a method `Export` that will save the person's details depending on the 'format' and 'location' parameters. This method was introduced, because the business required person details to be able to be exported say to Excel, JSON (for a webservice being implemented), etc...

```
public class Person {
    public string FirstName { get; set; }
    public string LastName { get; set; }    
    public DateTime DateOfBirth { get; set;}
    public char Gender { get; set; }
    public object Export(string format, string? location = null) {
        switch(format) {
            case "Excel":
                // excel specific rendering and saving to location here
                excel.Save(location);
                break;
            case "JSON":
                // json specific rendering and then returning that data as a result of our method/function
                return jsonString;
                break;
            default:
                // default behaviour here, saving it in a text file
                File.WriteAllText(location.Value, $"{FirstName} {LastName}, born on {DateOfBirth.ToShortDateString()} as {Gender}.");
        }
    }
}
```

#### Why it looks ugly
One of the first things we can see is that the class `Person` is responsible for exporting it's data to another format, this is a clear **violation of the Single Responsibility principle**. Say the API of Excel changes, one will need to go into the `Person` object and change the `Export` method logic of Excel changes, one would have to go into the `Export` method and change the details of the implementation. That change will most likely affect any class depending on the `Person` object.

Clearly there are also more things that  "*look ugly*", but we're focusing on the Single Responsibility principle here first and will deal with the other problems the current implementation of `Person` has now.

### How to fix?
First, we will extract the responsibility of exporting (aka the `Export` method) out of the `Person` class by introducing a `PersonExporter` class.

```
public class Person {
    public string FirstName { get; set; }
    public string LastName { get; set; }    
    public DateTime DateOfBirth { get; set;}
    public char Gender { get; set; }
}
```

```
public class PersonExporter {
    public object Export(string format, string? location = null) {
        switch(format) {
            case "Excel":
                // excel specific rendering and saving to location here
                excel.Save(location);
                break;
            case "JSON":
                // json specific rendering and then returning that data as a result of our method/function
                return jsonString;
                break;
            default:
                // default behaviour here, saving it in a text file
                File.WriteAllText(location.Value, $"{FirstName} {LastName}, born on {DateOfBirth.ToShortDateString()} as {Gender}.");
        }
    }
}
```

The `Person` class itself is now free of any other responsibility than keeping hold of data, yet we do have other issues with the `Export` method, itself holds a lot of responsibilities and other *code smells* that we will fix later on with the other principles.

#### Resources & Links
- https://code.tutsplus.com/tutorials/solid-part-1-the-single-responsibility-principle--net-36074
- http://www.oodesign.com/single-responsibility-principle.html
- http://softwareengineering.stackexchange.com/questions/342051/violation-solution-for-single-responsibility-principle
- https://app.pluralsight.com/library/courses/principles-oo-design/table-of-contents

## Open/Closed Principle
> Software entities (e.g. classes, modules, functions) should be open for extension but closed for modification.
- You should be able to extend/add functionaly by adding new classes and/or function while not changing the inners of your existing classes or functions.
- Design so that 
  - software is open to extension and new behaviour can be added in the future,
  - but closed to modification to changes to source or binary code are not required.

### Violation example
Building on from the `PersonExporter` class we introduced in the [*how to fix* of the Single Responsibility Principle](#how-to-fix) - the business decided they need to interface with an application that only accepts XML input. This means we'll have to change the ´Export´ class to support this new beahviour.

```
public class PersonExporter {
    public object Export(Person person, string format, string? location = null) {
        switch(format) {
            case "Excel":
                // excel specific rendering and saving to location here
                excel.Save(location);
                break;
            case "JSON":
                // json specific rendering and then returning that data as a result of our method/function
                return jsonString;
                break;
            case "XML":
                // xml specific rendering and then returning that data as a result of our method/function
                return xmlString;
            default:
                // default behaviour here, saving it in a text file
                File.WriteAllText(location.Value, $"{FirstName} {LastName}, born on {DateOfBirth.ToShortDateString()} as {Gender}.");
        }
    }
}
```

#### Why it looks ugly
Every time we want to introduce a new format, a change to the `PersonExporter` is required. This clearly violates the *open/closed* principle as it requires us to change an existing function to add new behaviour.

### How to fix?
A way to change behaviour/add new functionaly without changing preexisting code is to rely on abstractions. Here we will make an interface called `IPersonFormatter` and introduce that to the `Export` method as parameter. Each class inheriting from `IPersonFormatter` (e.g. `JSONPersonFormatter`, `XMLPersonFormatter`...) will then implement it's own logic in the `Format` method.

```
interface IPersonFormatter<TReturnValue> {
    TReturnValue Format(Person person, string? location);
}

public class JSONPersonFormatter : IPersonFormatter<string> {
    public string Format(Person person, string? location) {
        // ignoring location and executing json formatting logic
        return jsonString;
    }
}

public class XMLPersonFormatter : IPersonFormatter<string> {
    public string Format(Person person, string? location) {
        // ignoring location and executing xml formatting logic
        return xmlString;
    }
}

public class ExcelPersonFormatter : IPersonFormatter<object> {
    public object Format(Person person, string? location) {
        // excel specific rendering and saving to location here        
        excel.Save(location);
        // we return null, as we don't have anything to return...
        return null;
    }
}
```

```
public class PersonExporter {
    public object Export(Person person, IPersonFormatter formatter, string? location) {        
        return formatter.Format(person, location);
    }
}
```

This way we can now add new formats and implementations without having to change the inner workings of the `PersonExporter` class. There are still some flaws in the design, and we will improve on them later on with use of the other principles.

#### Resources & Links
- http://www.oodesign.com/open-close-principle.html
- https://app.pluralsight.com/library/courses/principles-oo-design/table-of-contents

## Liskov Substitution Principle
> In a computer program, if type *B* is a subtype of type *A* then objects of type *A* may be replaced with objects of type *B* without altering any of the desirable properties of type *A*. In essence it means that every subclass should be substitutable for their base class. 
- If it looks like a duck, quacks like a duck and walks like a duck, it's probably a duck;
  - unless the the other duck either needs batteries - then it's probably not the right abstractions
  - or unless the other duck sometimes relies on other ducks to fly, or flies on it'sown - then it's probably not the right abstraction as well
    - *will refer to this again in the "Violation Example" of our IPersonFormatter design*
- Child classes must not remove base class behaviour, nor violate base class invariants.
- Code using the abstractions should not know they are different from their base types.

### Violation example
The `IPersonFormatter` and in general the `Export` method on the `PersonExporter` class violate this principle as the calling code has to be aware whether or not a result will be returned. In the case of our JSON and XML formatters a string is returned, but in the EXCEL formatter *null*/nothing is returned... This is confusing, and certainly makes it that even if it does *quack, walk and looks like a duck* it certainly *doesn't fly like a duck*...

```
public class XMLPersonFormatter : IPersonFormatter<string> {
    public string Format(Person person, string? location) {
        // ignoring location and executing xml formatting logic
        return xmlString;
    }
}

public class ExcelPersonFormatter : IPersonFormatter<object> {
    public object Format(Person person, string? location) {
        // excel specific rendering and saving to location here        
        excel.Save(location);
        // we return null, as we don't have anything to return...
        return null;
    }
}
```

### How to fix?

#### Resources & Links
- http://www.oodesign.com/liskov-s-substitution-principle.html
- https://app.pluralsight.com/library/courses/principles-oo-design/table-of-contents

## Interface Segregation Principle
> No client should be forced to depend on methods it does not use, implicitely implying that a client should never be forced to implement an interface it doesn't use.

### Violation example
### How to fix?

#### Resources & Links
- http://www.oodesign.com/interface-segregation-principle.html
- https://app.pluralsight.com/library/courses/principles-oo-design/table-of-contents

## Dependency Inversion Principle
> High-level modules should not depend on low-level modules, both should depend on abstractions. Abstractions should not depend on details, details should depend on abstractions.

### Violation example
### How to fix?

#### Resources & Links
- http://www.oodesign.com/dependency-inversion-principle.html
- https://app.pluralsight.com/library/courses/principles-oo-design/table-of-contents