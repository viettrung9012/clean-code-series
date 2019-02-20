# Chapter 10 - Classes

## How to write a clean Class

Classes should follow the same rules as Functions we talked about in chapter 3.

- Be organised
- Have a meaningful name
- Have one responsibility only
- Be **SMALL**!!

## Class Organisation

Simply follow the standard Java conventions:

1. Public static constants
2. Private static constants
3. Private instance variables
4. Constructors
5. Public functions
6. Private utility functions right after the calling public method
7. Getters and setters at the end

```java
public class MyClass {

    public static final String MY_PUBLIC_CONST = "THIS IS A CONSTANT EVERYBODY CAN ACCESS";

    private static final String MY_PRIVATE_CONST = "UTILITY DELIMETER";

    private String name;
    private int age;

    public MyClass(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public MyClass() {
    }

    public void doSomething() {
        utilitySomething();
    }

    private void utilitySomething() {
    }

    public String getName() {
        return this.name;
    }
    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return this.age;
    }
    public void setAge() {
        this.age = age;
    }
}
```

The only exception for having instance variables and utility functions other than private is for test purposes.
If your tests are correctly organised, they will be in the same package as the classes you want to test. So, you can make the instance variables and utility functions protected or package accessible only, and then you'll be able to access them in your tests.
But, that should be for exceptions only, like dealing with legacy code. If your classes and tests are correctly organised, your instance variables and utilities must always be private. Don't loosen encapsulation!

## Classes should be SMALL !!

Like Functions, classes have two rules. First, they should be small. Second, they should be smaller than that.

The difference is, for Functions we measure size by counting physical lines, but for classes we count **responsibilities**.

```java
public class SuperDashboard {
    public Component getLastFocusedComponent()
    public void setLastFocused(Component lastFocused)
    public int getMajorVersionNumber()
    public int getMinorVersionNumber()
    public int getBuildNumber()
}
```
> "That looks small. I really did a good job writing this clean code!"

**Wrong!**

It has too many responsibilities.

There are two ways to identify that a class has too many responsibilities:

1. The name is too generic or ambiguous.
2. The class has two reasons to change. 1 reason to change = 1 responsibility.

### What is a good Class name?

A name of a class should describe what responsibilities it fulfills. If we cannot derive a concise name for a class, it's likely too large.

We should be able to write a brief decription of the class in about 25 words without using the words "if", "and", "or", or "but".
Looking at the `SuperDashboard` above, we could describe it as

> The `SuperDashboard` provides access to the component that last held the focus, and it also allows us to track the version and build numbers.

the first "and" is a hint that `SuperDashboard` has too many responsibilities.


### The Single Responsibility Principle

The Single Responsibility Principle (SRP) states that a class or module should have one, and only one, **reason to change**.
This principle gives us both __*a definition of responsibility*__, and __*a guideline for class size*__.
Classes should have one responsibility - one reason to change.

A fix for the `SuperDashboard` would be to externalise the methods that deal with version information to another class:

```java
public class Version {
    public int getMajorVersionNumber()
    public int getMinorVersionNumber()
    public int getBuildNumber()
}
```

#### Why is SRP almost never followed?

- Developers think they're done once the program works
- Many fear that a large number of classes makes it more difficult to understand the bigger picture

But, do you want your tools organised into toolboxes with many small drawers or just toss everything into one box?

### Cohesion

Another way of keeping a class small is to make sure it is highly cohesive. It means that all the methods of a class should use all of its instance variables.

E.g. if a class has 5 instance variables and half the functions use the same 2 variables and the other half the other 3 remaining variables. Then, you can split this class into two with respectively 3 and 2 variables and the functions using them.

```java
class LowCohesion {
    private int _element1;
    private int _element2;
    private int _element3;

    public int MethodA() {
        var returnValue = SomeOtherMethod(_element1);
        return SomeVeryOtherMethod(_element2);
    }

    public void MethodB() {
        SomeOtherMethod(_element3);
    }

    public void PrintValues() {
        Console.WriteLine(_element1);
        Console.WriteLine(_element2);
    }
}
```

Refactored to:

```java
class HighCohesion1 {
    private int _element1;
    private int _element2;

    public int MethodA() {
        var returnValue = SomeOtherMethod(_element1);
        return SomeVeryOtherMethod(_element2);
    }

    public void PrintValues() {
        Console.WriteLine(_element1);
        Console.WriteLine(_element2);
    }
}

class HighCohesion2 {
    private int _element3;

    public void MethodB() {
        SomeOtherMethod(_element3);
    }
}
```

Having a class fully cohesive is almost impossible neither advisable. The focus here is to keep the cohesion high. **When classes lose cohesion, split them!**


## Organise for Change

Keeping Classes and Functions small might increase the number of classes, packages and lines of code that make your program or system. But, **it reduces the risk of change**.

Organising your classes by following the Single Responsibility Principle will:

- Reduce the chance that a change impact other parts of the system
- Reduce the comprehension time needed to understand a Class
- Make a system more flexible by increasing code reusability and reducing code duplication.
- Improve collaboration between developers by limiting change conflicts

Other ways of making your classes small and your system change-proof is to follow some patterns like Strategy and to use Interfaces. By doing so, you will isolate the concrete implementation details from the the concepts.