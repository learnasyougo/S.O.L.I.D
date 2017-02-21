# S.O.L.I.D: The First 5 Principles of Object Oriented Design.
 - [Single Responsibility Principle](#single-responsibility-principle)
 - [Open/Closed Principle](#openclosed-principle)
 - [Liskov Substitution Principle](#liskov-substitution-principle)
 - [Interface Segregation Principle](#interface-segragation-principle)
 - [Dependency Inversion Principle](#dependency-inversion-principle)
 - [Don't Repeat Yourself (DRY)](#dont-repeat-yourself-dry)


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
One possible change, there could be other solutions, would be to make a more reliable return value for the `Format` method. So that `IPersonFormatter`'s `Format` method always returns a `stream` (or another in memory form) of the data. We will also omit the location of the `Format` method, because it actually had nothing to do (considering the **single responsibility principle*) with formatting a Person.

```
IPersonFormatter {
    Stream Format(Person person);
}
```

```
public class XMLPersonFormatter : IPersonFormatter {
    public string Format(Person person) {
        // ignoring location and executing xml formatting logic
        // saving it to a MemoryStream when working in .NET for example
        return xmlStringMemoryStream;
    }
}

public class ExcelPersonFormatter : IPersonFormatter {
    public object Format(Person person) {
        // excel specific rendering and saving to location here        
        excel.Save(location);
        // we now don't return null, but a MemoryStream we don't have anything to return...
        return excelWorkBookMemoryStream;
    }
}
```

As now all implementations of `IPersonFormatter` return a predicatable `Stream` object, we can change the design of the `PersonExporter` class so that it handles the streams based on a given location. 

```
public class PersonExporter {
    public Stream Export(Person person, IPersonFormatter formatter, string? location) {        
        var stream = formatter.Format(person);
        if(location.HasValue) {
            // Handle saving the stream to given location/file            
        }
        // we return the stream - even if we already saved it
        return stream;
    }
}
```

Imagine some class or controller/api using this.
```
public class SomeController {
    public void ExportUsers() {
        var personExporter = new PersonExporter();
        var personFormatter = new JSONPersonFormatter();
        foreach(var person in _personRepository.GetUsers()) {
            var stream = personExporter.Export(person, personFormatter);
            // decided what to do with the returned stream depending on your requirements
            // save it to a list and return or do sth else
        }        
    }
}
```

This is already better, because we always return a stream, even in the `PersonExporter` `Export` method - but depending on the use case and design philosophy still feels a little funky as we're still stuck with the `location` parameter that handles saving a stream for us...
We coud abstract that further away too ofcourse, but in my opinion **pragmatism** comes into play here. How far do you abstract away? Is it a core design problem? Can we refactor later on in the process? So you could leave it here, although I would omit the location as it probably would be better to seperate it to a dedicated seperated module that handles saving streams to file. But that would probably lead us to far from the topic at hand here.

#### Resources & Links
- http://www.oodesign.com/liskov-s-substitution-principle.html
- https://app.pluralsight.com/library/courses/principles-oo-design/table-of-contents

## Interface Segregation Principle
> No client should be forced to depend on methods it does not use, implicitely implying that a client should never be forced to implement an interface it doesn't use.
- Prefer small cohesive interfaces, to large bulky interfaces.

### Violation example
Usually when you have large and bulky interfaces, for example here's an interface for a list of people and an implementation of it that's only interested in reading, as it's used on for example the frontend of an information display.

```
public interface IPersonRepository {
    void Save(Person person);
    void Save(IEnumerable<Person> people);
    Person Get(int id);
    IEnumerable<Person> Get();
    IEnumerable<Person> Get(Func<Person, bool> predicate);
}

public class PersonReadOnlyRepository : IPersonRepository {
    public void Save(Person person) {
        throw NotImplementedException();
    }
    public void Save(IEnumerable<Person> people) {
        throw NotImplementedException();
    }
    public Person Get(int id) {
        throw NotImplementedException();
    }
    public IEnumerable<Person> Get() {
        using(var db = new DbContext()) {
            return db.People.ToEnumerable();
        }
    }
    public IEnumerable<Person> Get(Func<Person, bool> predicate) {
        using(var db = new DbContext()) {
            return db.People.Where(p => predicate(p)).ToEnumerable();
        }        
    }
}
```

The methods we're not interested in are now throwing the `NotImplementedException` - which in it's turn violates the **Liskov Subsition Principle**. Thus, we have a problem at hand.

### How to fix?
Seperate a larger bulky interface into smaller interfaces.

```
public interface IWriteablePersonRepository {
    void Save(Person person);
    void Save(IEnumerable<Person> people);
}

public interface IReadablePersonRepository {
    Person Get(int id);
    IEnumerable<Person> Get();
    IEnumerable<Person> Get(Func<Person, bool> predicate);
}

public class PersonReadOnlyRepository : IReadablePersonRepository {
    public Person Get(int id) {
        using(var db = new DbContext()) {
            return db.People.GetById(id);
        }
    }
    public IEnumerable<Person> Get() {
        using(var db = new DbContext()) {
            return db.People.ToEnumerable();
        }
    }
    public IEnumerable<Person> Get(Func<Person, bool> predicate) {
        using(var db = new DbContext()) {
            return db.People.Where(p => predicate(p)).ToEnumerable();
        }        
    }
}
```

As long as you keep interfaces small enough, you can easily expose larger implementations to the client, but with a limited interface. In the example above we created a special `PersonReadOnlyRepository`, but consider the example below. There we make a full blown `PersonRepository` which implements both `IWriteablePersonRepository` and `IReadablePersonRepository`. When passing the object to the `ViewClient` class's constructor, the argument expected there is an IReadablePersonRepository - thus, limiting the functionality in the `ViewClient`.

```
public class PersonRepository : IReadablePersonRepository, IWriteablePersonRepository {
    public void Save(Person person) {
        using(var db = new DbContext()) {
            return db.People.Add(person);
        }
    }
    public void Save(IEnumerable<Person> people) {
        using(var db = new DbContext()) {
            return db.People.AddRange(people);
        }
    }
    public Person Get(int id) {
        using(var db = new DbContext()) {
            return db.People.GetById(id);
        }
    }
    public IEnumerable<Person> Get() {
        using(var db = new DbContext()) {
            return db.People.ToEnumerable();
        }
    }
    public IEnumerable<Person> Get(Func<Person, bool> predicate) {
        using(var db = new DbContext()) {
            return db.People.Where(p => predicate(p)).ToEnumerable();
        }        
    }
}

public class ViewClient {
    private readonly _personRepository;

    public ViewClient(IReadablePersonRepository personRepository) {
        _personRepository = personRepository;
    }
}
```

#### Resources & Links
- http://www.oodesign.com/interface-segregation-principle.html
- https://app.pluralsight.com/library/courses/principles-oo-design/table-of-contents

## Dependency Inversion Principle
> High-level modules should not depend on low-level modules, both should *depend* on abstractions. Abstractions should not *depend* on details, details should depend on abstractions.
- Class constructors, or (public/proteced) methods, should require any dependencies. And let you know what they need to do it's work.

The **dependencies** we talk about here are for example the **new** keyboard (creating instances), static methods, third party libraries, database(s), file system, configuration, ftp, web services, system resources etc.. Thus some violations could be:
- High level modules usually call low level modules, and thus mostly tend to *instantiate* them (using the **new** keyword) - not what we want. 
- Static methods for example used as `Façade` layers.

### Violation example
In the `PersonRepository` class we made above we have a default empty constructor and in our methods we just assume the use of given `DbContext` by instantiating a new object of it every time. This not only violates the `Dependency Inversion principle`, but also the `Single Responsibility principle` (it is responsible for creating new `DbContext` instances) as well as the `Open/Closed principle` (when we want to change `DbContext` to another implementation, we have to digg in and **modify** the class, not what we want).

```
public class PersonRepository : IReadablePersonRepository, IWriteablePersonRepository {
    public void Save(Person person) {
        using(var db = new DbContext()) {
            return db.People.Add(person);
        }
    }
    public void Save(IEnumerable<Person> people) {
        using(var db = new DbContext()) {
            return db.People.AddRange(people);
        }
    }
    public Person Get(int id) {
        using(var db = new DbContext()) {
            return db.People.GetById(id);
        }
    }
    public IEnumerable<Person> Get() {
        using(var db = new DbContext()) {
            return db.People.ToEnumerable();
        }
    }
    public IEnumerable<Person> Get(Func<Person, bool> predicate) {
        using(var db = new DbContext()) {
            return db.People.Where(p => predicate(p)).ToEnumerable();
        }        
    }
}
```

Other **code smells**
- using the **new** keyword, creating new instances
- use of static methods or properties, e.g.: ´DateTime.Now´ implemented in your method in stead of the method accepting a `DateTime` object and then using that one.

### How to fix?
By introducting a constructor with a parameter `IDbContext`. Now when using PersonRepository, the class is honest about it's dependency to `IDbContext`.

```
public class PersonRepository : IReadablePersonRepository, IWriteablePersonRepository {
    private readonly _db;

    public PersonRepository(IDbContext db) {
        _db = db;
    }

    public void Save(Person person) {
        _db.People.Add(person);        
    }
    public void Save(IEnumerable<Person> people) {
        _db.People.AddRange(people);        
    }
    public Person Get(int id) {
        _db.People.GetById(id);        
    }
    public IEnumerable<Person> Get() {
        _db.People.ToEnumerable();        
    }
    public IEnumerable<Person> Get(Func<Person, bool> predicate) {
        _db.People.Where(p => predicate(p)).ToEnumerable();                
    }
}
```

**But where do I instantiate PersonRepository etc... when I can't use new?**
- a default constructor (poor man's *Inversion of Control*)
- main method where you instantiate those
- use an *Inversion of Control* container / framework

#### Resources & Links
- http://www.oodesign.com/dependency-inversion-principle.html
- https://app.pluralsight.com/library/courses/principles-oo-design/table-of-contents

## Don't Repeat Yourself (DRY)
> Every piece of knowledge must have a single, unambiguous, authorative representation within a system.
- In code
- In process
- In documentation

### Violations
Repetition in code, aka "*WET*", as in "*write everything twice*" or "*waste everyone's time*".

#### Magic strings & numbers
- E.g. 
  - a connection string 
    - store the value in a config file for example
    - make sure that you make it a class level variable    
    - and maybe/optionally accept it in the constructor and use dependency inversion
  - "*magic numbers*" => such as in if-conditions (`if(a > 1) { }`)
    - make sure to use refactoring to extract methods
    - adapt the magic numbers to parameters or varibales e.g. a > list.Count()
    - maybe make a local field and use the field in the highest scope as possible
    
etc...

#### Resources & Links
- https://app.pluralsight.com/library/courses/principles-oo-design/table-of-contents
- https://en.wikipedia.org/wiki/Don't_repeat_yourself

# Watch "Refactoring to a S.O.L.I.D. Foundation", Steve Bohlen
https://www.youtube.com/watch?v=huEEkx5P5Hs
<iframe width="853" height="480" src="https://www.youtube.com/embed/huEEkx5P5Hs" frameborder="0" allowfullscreen></iframe>
