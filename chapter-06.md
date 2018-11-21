# Chapter 6 - Objects and Data Structures

## Data Abstraction:

Hiding implementation is about abstractions! A class does not simply push its variables out through getters and setters. Rather it exposes abstract interfaces that allow its users to manipulate the essence of the data, without having to know its implementation.

```java
public class Point {
    public double x;
    public double y;
}

public interface Point {
    double getX();
    double getY();
    void setCartesian(double x, double y);
    double getR();
    double getTheta();
    void setPolar(double r, double theta);
}

public interface Vehicle {
    double getFuelTankCapacityInGallons();
    double getGallonsOfGasoline();
}

public interface Vehicle {
    double getPercentFuelRemaining();
}
```

## Data/Object Anti-Symmetry:

Objects hide their data behind abstractions and expose functions that operate on that data. Data struc- ture expose their data and have no meaningful functions.

```java
/* Example: Procedural Shape */
public class Square {
    public Point topLeft;
    public double side;
}
public class Rectangle {
    public Point topLeft;
    public double height;
    public double width;
}
public class Circle {
    public Point center;
    public double radius;
}
public class Geometry {
    public final double PI = 3.141592653589793;

    public double area(Object shape) throws NoSuchShapeException {
        if (shape instanceof Square) {
            Square s = (Square) shape;
            return s.side * s.side;
        } else if (shape instanceof Rectangle) {
            Rectangle r = (Rectangle) shape;
            return r.height * r.width;
        } else if (shape instanceof Circle) {
            Circle c = (Circle) shape;
            return PI * c.radius * c.radius;
        }
        throw new NoSuchShapeException();
    }
}

/* Example: Ploygraphic Shape */
public class Square implements Shape {
    private Point topLeft;
    private double side;
    public double area() {
        return side * side;
    }
}
public class Rectangle implements Shape {
    private Point topLeft;
    private double height;
    private double width;
    public double area() {
        return height * width;
    }
}
public class Circle implements Shape {
    private Point center;
    private double radius;
    public final double PI = 3.141592653589793;
    public double area() {
        return PI * radius * radius;
    }
}
```

Procedural code (code using data structures) makes it easy to add new functions without changing the existing data structures. OO code, on the other hand, makes it easy to add new classes without changing existing functions.

The complement is also true:

Procedural code makes it hard to add new data structures because all the functions must change. OO code makes it hard to add new functions because all the classes must change.

## The Law of Demeter:

The Law of Demeter says that a method f of a class C should only call the methods of these:
- C
- An object created by f
- An object passed as an argument to f
- An object held in an instance variable of C

An example for violating law of Demeter could be:

```java
final String outputDir = ctxt.getOptions().getScratchDir().getAbsolutePath();
```
This kind of code is often called a train wreck because it look like a bunch of coupled train cars. Chains of calls like this are generally considered to be sloppy style and should be avoided. It is usually best to split them up as follows:

```java
Options opts = ctxt.getOptions();
File scratchDir = opts.getScratchDir();
final String outputDir = scratchDir.getAbsolutePath();
```
Whether this is a violation of Demeter depends on whether or not ctxt, Options, and ScratchDir are objects or data structures.

```java
final String outputDir = ctxt.options.scratchDir.absolutePath;
```

This issue would be a lot less confusing if data structures simply had public variables and no functions, whereas objects had private variables and public functions. However, there are frameworks and standards (e.g., “beans”) that demand that even simple data structures have accessors and mutators.

## Hybrid

This confusion sometimes leads to unfortunate hybrid structures that are half object and half data structure. They have functions that do significant things, and they also have either public variables or public accessors and mutators that, for all intents and purposes, make the private variables public, tempting other external functions to use those variables the way a procedural program would use a data structure.

Such hybrids make it hard to add new functions but also make it hard to add new data structures. They are the worst of both worlds. Avoid creating them. They are indicative of a muddled design whose authors are unsure of—or worse, ignorant of—whether they need protection from functions or types.

Alternatively,
```java
ctxt.getAbsolutePathOfScratchDirectoryOption();
```

or
```java
ctxt.getScratchDirectoryOption().getAbsolutePath()
```

The first option could lead to an explosion of methods in the ctxt object. The second pre- sumes that getScratchDirectoryOption() returns a data structure, not an object.

If ctxt is an object, we should be telling it to do something; we should not be asking it about its internals.

### Intension:
```java
String outFile = outputDir + "/" + className.replace('.', '/') + ".class"; 
FileOutputStream fout = new FileOutputStream(outFile); 
BufferedOutputStream bos = new BufferedOutputStream(fout);
```

Asking ctxt object to do above would help us achieve Law of Demeter. Like below:

```java
BufferedOutputStream bos = ctxt.createScratchFileStream(classFileName);
```

## Data Transfer Objects: 

The data structure is a class with public variable and no functions. These are sometime called data transfer objects. These are widely used between communications between databases or parsing messages from sockets.

Term “bean” is somewhat common. Beans have private variables manipulated by getters and setters.  

```java
public class Address {
    private String street;
    private String streetExtra;
    private String city;
    private String state;
    private String zip;
    public Address(String street, String streetExtra, String city, String state, String zip) {
        this.street = street;
        this.streetExtra = streetExtra;
        this.city = city;
        this.state = state;
        this.zip = zip;
    }
    public String getStreet() {
        return street;
    }
    public String getStreetExtra() {
        return streetExtra;
    }
    public String getCity() {
        return city;
    }
    …

```

## Active Record: 

Active Records are special forms of DTOs. They are data structures with public (or bean- accessed) variables; but they typically have navigational methods like save and find. Typi- cally these Active Records are direct translations from database tables, or other data sources.

## Conclusion:

Objects expose behavior and hide data. This makes it easy to add new kinds of objects without changing existing behaviors. It also makes it hard to add new behaviors to existing objects. Data structures expose data and have no significant behavior. This makes it easy to add new behaviors to existing data structures but makes it hard to add new data structures to existing functions.

In any given system we will sometimes want the flexibility to add new data types, and so we prefer objects for that part of the system. Other times we will want the flexibility to add new behaviors, and so in that part of the system we prefer data types and procedures. Good software developers understand these issues without prejudice and choose the approach that is best for the job at hand.
