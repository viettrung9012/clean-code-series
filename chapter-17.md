# Chapter 17 - Smells and Heuristics

This list was compiled by walking through several different programs and refactoring them. 

This list is meant to be read from top to bottom and also to be used as a reference.

## Comments

**C1: Inappropriate Information**

It is inappropriate for a comment to hold information better held in a different kind of system such as:
 - source code control system
 - issue tracking system
 - any other record-keeping system
 
Change histories, for example, just clutter up source files with volumes of historical and uninteresting text. 

In general, meta-data as followings should not appear in comments as well:
 - authors
 - last-modified-date
 
Comments should be reserved for technical notes about the code and design.

**C2: Obsolete Comment**

A comment that has gotten old, irrelevant, and incorrect is obsolete.

- It is best not to write a comment that will become obsolete. 
- It is best to update it or get rid of it as quickly as possible if you find an obsolete comment.
- Obsolete comments tend to migrate away from the code they once described.

If we keep the obsolete comments, they may cause confusion and misdirect developers who are referencing the code.

**C3: Redundant Comment**

A comment is redundant if it describes something that adequately describes itself.

****Example1:****

```
i++; // increment i
```
****Example2:****
```
/**
* @param sellRequest
* @return
* @throws ManagedComponentException */
public SellResponse beginSellItem(SellRequest sellRequest) throws ManagedComponentException
```

**C4: Poorly Written Comment**

Make sure it is the best comment you can write. 

Choose your words carefully. Use correct grammar and punctuation. Don’t ramble. Don’t state the obvious. Be brief.

**C5: Commented-Out Code**

Commented-out code is an abomination. When you see commented-out code, delete it!

## Environment

**E1: Build Requires More Than One Step**

You should be able to check out the system with one simple command and then issue one other simple command to build it.

****Example****
```bash
svn get mySystem 
cd mySystem
ant all
```

**E2: Tests Require More Than One Step**

You should be able to run all the unit tests with just one command. 

In the best case you can run all the tests by clicking on one button in your IDE. In the worst case you should be able to issue a single simple command in a shell.

## Functions

**F1: Too Many Arguments**

Functions should have a small number of arguments. No argument is best, followed by one, two, and three. More than three is very questionable and should be avoided with prejudice.

**F2: Output Arguments**

Output arguments are counterintuitive. Readers expect arguments to be inputs, not outputs. If your function must change the state of something, have it change the state of the object it is called on.

**F3: Flag Arguments**

Boolean arguments loudly declare that the function does more than one thing. They are
confusing and should be eliminated. 

**F4: Dead Function**

Methods that are never called should be discarded. Keeping dead code around is wasteful. Don’t be afraid to delete the function. Remember, your source code control system still remembers it.

## General

**G1: Multiple Languages in One Source File**

- Programming environments make it possible to put many different languages into a single source file.
- Take pains to minimize both the number and extent of extra languages in our source files.

**G2: Obvious Behavior Is Unimplemented**

The Principle of Least Surprise: any function or class should implement the behaviors that another programmer could reasonably expect.

`Day day = DayDate.StringToDay(String dayName);`

**G3: Incorrect Behavior at the Boundaries**

- We seldom realize just how complicated correct behavior is.
- We often write functions that we think will work.
- We trust our intuition rather than trying to prove our code works in all the corner and boundary cases.

Don’t rely on your intuition. Look for every boundary condition and write a test for it.

**G4: Overridden Safeties**

Turning off failing tests and telling yourself you’ll get them to pass later is as bad as pretending your credit cards are free money.

**G5: Duplication**

1. Every time you see duplication in the code, it represents a missed opportunity for abstraction.
2. By folding the duplication into such an abstraction, you increase the vocabulary of the language of your design.
3. Other programmers can use the abstract facilities you create.
4. Coding becomes faster and less error prone because you have raised the abstraction level.

Find and eliminate duplication wherever you can.

**G6: Code at Wrong Level of Abstraction**

- Create abstractions that separate higher level general concepts from lower level detailed concepts.
- Create abstract classes to hold the higher level concepts and derivatives to hold the lower level concepts.
- Make sure the separation is complete.

__e.g. constants, variables, utility functions__

Good software design requires that we separate concepts at different levels and place them in different containers. Sometimes these containers are base classes or derivatives and sometimes they are source files, modules, or components. 

**G7: Base Classes Depending on Their Derivatives**

- Base classes should know nothing about their derivatives.
- We want to be able to deploy derivatives and bases in different jar files.
- This allows us to deploy our systems in discrete and independent components.

When such components are modified, they can be redeployed without having to redeploy the base components. This means that the impact of a change is greatly lessened, and maintaining systems in the field is made much simpler.

**G8: Too Much Information**

Well-defined modules:
- have very small interfaces that allow you to do a lot with a little.
- A well-defined interface does not offer very many functions to depend upon, so coupling is low.

Poorly defined modules:
- have wide and deep interfaces that force you to use many different gestures to get simple things done.
- A poorly defined interface provides lots of functions that you must call, so coupling is high.

Good software developers learn to limit what they expose at the interfaces of their classes and modules. The fewer methods a class has, the fewer variables a function knows about, the fewer instance variables a class has, the better.

**G9: Dead Code**

Dead code is code that isn’t executed. You find it in:
1. the body of an `if` statement that checks for a condition that can’t happen.
2. the `catch` block of a `try` that never `throws`.
3. little utility methods that are never called or `switch/case` conditions that never occur.

Dead code is not completely updated when designs change. It still compiles, but it does not follow newer conventions or rules. Delete it from the system.

**G10: Vertical Separation**

Variables and function should be defined close to where they are used.

- Local variables should be declared just above their first usage and should have a small vertical scope.
- Private functions should be defined just below their first usage.

**G11: Inconsistency**

Be careful with the conventions you choose, and once chosen, be careful to continue to follow them.

If within a particular function you use a variable named `response` to hold an `HttpServletResponse`, then use the same variable name consistently in the other functions that use `HttpServletResponse` objects. If you name a method `processVerificationRequest`, then use a similar name, such as `processDeletionRequest`, for the methods that process other kinds of requests.

**G12: Clutter**

Variables that aren’t used, functions that are never called, comments that add no information, and so forth. All these things are clutter and should be removed. Keep your source files clean, well organized, and free of clutter.

**G13: Artificial Coupling**

Things that don’t depend upon each other should not be artificially coupled.

In general an artificial coupling is a coupling between two modules that serves no direct purpose. It is a result of putting a variable, constant, or function in a temporarily convenient, though inappropriate, location.

Take the time to figure out where functions, constants, and variables ought to be declared. Don’t just toss them in the most convenient place at hand and then leave them there.

**G14: Feature Envy**

The methods of a class should be interested in the variables and functions of the class they belong to, and not the variables and functions of other classes.

When a method uses accessors and mutators of some other object to manipulate the data within that object, then it envies the scope of the class of that other object. It wishes that it were inside that other class so that it could have direct access to the variables it is manipulating.

```java
public class HourlyPayCalculator {
    public Money calculateWeeklyPay(HourlyEmployee e) {
        int tenthRate = e.getTenthRate().getPennies();
        int tenthsWorked = e.getTenthsWorked();
        int straightTime = Math.min(400, tenthsWorked);
        int overTime = Math.max(0, tenthsWorked - straightTime);
        int straightPay = straightTime * tenthRate;
        int overtimePay = (int)Math.round(overTime*tenthRate*1.5);
        return new Money(straightPay + overtimePay);
    }
}
```

All else being equal, we want to eliminate Feature Envy because it exposes the internals of one class to another.

```java
public class HourlyEmployeeReport {
    private HourlyEmployee employee;

    public HourlyEmployeeReport(HourlyEmployee e) {
        this.employee = e;
    }

    String reportHours() {
        return String.format(
            "Name: %s\tHours:%d.%1d\n",
            employee.getName(),
            employee.getTenthsWorked()/10,
            employee.getTenthsWorked()%10);
    }
}
```

**G15: Selector Arguments**

```java
public int calculateWeeklyPay(boolean overtime) {
    int tenthRate = getTenthRate();
    int tenthsWorked = getTenthsWorked();
    int straightTime = Math.min(400, tenthsWorked);
    int overTime = Math.max(0, tenthsWorked - straightTime);
    int straightPay = straightTime * tenthRate;
    double overtimeRate = overtime ? 1.5 : 1.0 * tenthRate;
    int overtimePay = (int)Math.round(overTime*overtimeRate);
    return straightPay + overtimePay;
}
```

Not only is the purpose of a selector argument difficult to remember, each selector argument combines many functions into one. Selector arguments are just a lazy way to avoid splitting a large function into several smaller functions.

```java
public int straightPay() {
    return getTenthsWorked() * getTenthRate();
}

public int overTimePay() {
    int overTimeTenths = Math.max(0, getTenthsWorked() - 400);
    int overTimePay = overTimeBonus(overTimeTenths);
    return straightPay() + overTimePay;
}

private int overTimeBonus(int overTimeTenths) {
    double bonus = 0.5 * getTenthRate() * overTimeTenths;
    return (int) Math.round(bonus);
}
```

In general it is better to have many functions than to pass some code into a function to select the behavior.

**G16: Obscured Intent**

```java
public int m_otCalc() {
    return iThsWkd * iThsRte +
        (int) Math.round(0.5 * iThsRte *
            Math.max(0, iThsWkd - 400)
    );
}
```

Small and dense as this might appear, it’s also virtually impenetrable. It is worth taking the time to make the intent of our code visible to our readers.

**G17: Misplaced Responsibility**

Code should be placed where a reader would naturally expect it to be.

Sometimes we get “clever” about where to put certain functionality. We’ll put it in a function that’s convenient for us, but not necessarily intuitive to the reader.

```
                total worked hours
                   /           \
                  /             \
                 /               \
                /                 \
               /                   \
        report module        timecard module
        getTotalHours()?     saveTimeCard()?
```

**G18: Inappropriate Static**

`HourlyPayCalculator.calculatePay(employee, overtimeRate);`

This seems like a reasonable `static` function. It doesn’t operate on any particular object and gets all it’s data from it’s arguments. However, there is a reasonable chance that we’ll want this function to be polymorphic. We may wish to implement several different algorithms for calculating hourly pay.

- In general you should prefer nonstatic methods to static methods.
- When in doubt, make the function nonstatic.
- If you really want a function to be static, make sure that there is no chance that you’ll want it to behave polymorphically.

**G19: Use Explanatory Variables**

One of the more powerful ways to make a program readable is to break the calculations up into well-named intermediate values.

```java
Matcher match = headerPattern.matcher(line);
if (match.find()) {
    String key = match.group(1);
    String value = match.group(2);
    headers.put(key.toLowerCase(), value);
}
```

More explanatory variables are generally better than fewer.

**G20: Function Names Should Say What They Do**

`Date newDate = date.add(5);`

- If the function adds five days to the date and changes the date, then it should be called `addDaysTo` or `increaseByDays`.
- If the function returns a new date that is five days later but does not change the date instance, it should be called `daysLater` or `daysSince`.

**G21: Understand the Algorithm**

Before you consider yourself to be done with a function, make sure you understand how it works. It is not good enough that it passes all the tests. You must know that the solution is correct.

Often the best way to gain this knowledge and understanding is to refactor the function into something that is so clean and expressive that it is obvious how it works.

**G22: Make Logical Dependencies Physical**

1. If one module depends upon another, that dependency should be physical, not just logical.
2. The dependent module should not make assumptions (in other words, logical dependencies) about the module it depends upon.
3. Rather it should explicitly ask that module for all the information it depends upon.

```java
public class HourlyReporter {
    private HourlyReportFormatter formatter;
    private List<LineItem> page;
    private final int PAGE_SIZE = 55;

    public HourlyReporter(HourlyReportFormatter formatter) {
        this.formatter = formatter;
        page = new ArrayList<LineItem>();
    }

    public void generateReport(List<HourlyEmployee> employees) {
        for (HourlyEmployee e : employees) {
            addLineItemToPage(e);
            if (page.size() == PAGE_SIZE)
                printAndClearItemList();
        }
        if (page.size() > 0)
            printAndClearItemList();
    }

    private void printAndClearItemList() {
        formatter.format(page);
        page.clear();
    }

    private void addLineItemToPage(HourlyEmployee e) {
        LineItem item = new LineItem();
        item.name = e.getName();
        item.hours = e.getTenthsWorked() / 10;
        item.tenths = e.getTenthsWorked() % 10;
        page.add(item);
    }

    public class LineItem {
        public String name;
        public int hours;
        public int tenths;
    }
}
```

Why should the `HourlyReporter` know `PAGE_SIZE`? Page size should be the responsibility of the `HourlyReportFormatter`.

That causes `HourlyReporter` to assume that it knows what the page size ought to be. Such an assumption is a logical dependency.

**G23: Prefer Polymorphism to If/Else or Switch/Case**

Consider polymorphism before using a switch.

__ONE SWITCH__ rule: There may be no more than one `switch` statement for a given type of selection. The cases in that `switch` statement must create polymorphic objects that take the place of other such `switch` statements in the rest of the system.

**G24: Follow Standard Conventions**

1. Every team should follow a coding standard based on common industry norms.
2. This coding standard should specify things like
    1. where to declare instance variables;
    2. how to name classes, methods, and variables;
    3. where to put braces; and so on.
3. The team should not need a document to describe these conventions because their code provides the examples.

**G25: Replace Magic Numbers with Named Constants**

In general it is a bad idea to have raw numbers in your code. You should hide them behind well-named constants.

```
final int SECONDS_PER_DAY = 86400;
final int LINES_PER_PAGE = 55;
```

But some constants are so easy to recognize that they don’t always need a named constant to hide behind so long as they are used in conjunction with very self-explanatory code.

```
double milesWalked = feetWalked/5280.0;
int dailyPay = hourlyRate * 8;
double circumference = radius * Math.PI * 2;
```

Do we really need the constants `FEET_PER_MILE`, `WORK_HOURS_PER_DAY`, and `TWO` in the above examples?

```
double Pi = 3.141592653589793;
double Pi = 3.1415927535890793;
```

Did you catch the single-digit error? Therefore, it is a good thing that `Math.PI` has already been defined for us.

The term “Magic Number” does not apply only to numbers. It applies to any token that has a value that is not self-describing.

```
assertEquals(7777, Employee.find(“John Doe”).employeeNumber());
```

```
assertEquals(
    HOURLY_EMPLOYEE_ID,
    Employee.find(HOURLY_EMPLOYEE_NAME).employeeNumber());
```

**G26: Be Precise**

- When you make a decision in your code, make sure you make it precisely.
- Know why you have made it and how you will deal with any exceptions.
- Don’t be lazy about the precision of your decisions.

If you,
1. decide to call a function that might return null, make sure you check for null.
2. query for what you think is the only record in the database, make sure your code checks to be sure there aren’t others.
3. need to deal with currency, use integers and deal with rounding appropriately.
4. think there is the possibility of concurrent update, make sure you implement some kind of locking mechanism.

**G27: Structure over Convention**

Enforce design decisions with structure over convention.

e.g. switch/cases with nicely named enumerations are inferior to base classes with abstract methods.

**G28: Encapsulate Conditionals**

Boolean logic is hard enough to understand without having to see it in the context of an `if` or `while` statement. Extract functions that explain the intent of the conditional.

`if (shouldBeDeleted(timer))`

`if (timer.hasExpired() && !timer.isRecurrent())`

**G29: Avoid Negative Conditionals**

Negatives are just a bit harder to understand than positives. So, when possible, conditionals should be expressed as positives.

`if (buffer.shouldCompact())`

`if (!buffer.shouldNotCompact())`

**G30: Functions Should Do One Thing**

```
                           ---------------------------------------
                           | functions with multiple sections    |
                           | that perform a series of operations |
                           ---------------------------------------
                            /                 |                 \
                           /                  |                  \
                          /                   |                   \
                         /                    |                    \
               smaller function        smaller function        smaller function
               with one thing          with one thing          with one thing

```

```java
public void pay() {
    for (Employee e : employees) {
        if (e.isPayday()) {
            Money pay = e.calculatePay();
            e.deliverPay(pay);
        }
    }
}
```

```java
public void pay() {
    for (Employee e : employees)
        payIfNecessary(e);
}

private void payIfNecessary(Employee e) {
    if (e.isPayday())
        calculateAndDeliverPay(e);
}

private void calculateAndDeliverPay(Employee e) {
    Money pay = e.calculatePay();
    e.deliverPay(pay);
}
```

**G31: Hidden Temporal Couplings**

Temporal couplings are often necessary, but you should not hide the coupling. Structure the arguments of your functions such that the order in which they should be called is obvious.

```java
public class MoogDiver {
    Gradient gradient;
    List<Spline> splines;

    public void dive(String reason) {
        saturateGradient();
        reticulateSplines();
        diveForMoog(reason);
    }
    ...
}
```

Unfortunately, this code does not enforce temporal coupling.

```java
public class MoogDiver {
    Gradient gradient;
    List<Spline> splines;

    public void dive(String reason) {
        Gradient gradient = saturateGradient();
        List<Spline> splines = reticulateSplines(gradient);
        diveForMoog(splines, reason);
    }
    ...
}
```

This exposes the temporal coupling by creating a bucket brigade. Each function produces a result that the next function needs, so there is no reasonable way to call them out of order.

**G32: Don’t Be Arbitrary**

- If a structure appears arbitrary, others will feel empowered to change it.
- If a structure appears consistently throughout the system, others will use it and preserve the convention.

```
public class AliasLinkWidget extends ParentWidget {
    public static class VariableExpandingWidgetRoot {
        ...
    ...
}
```

Public classes that are not utilities of some other class should not be scoped inside another class. The convention is to make them public at the top level of their package.

**G33: Encapsulate Boundary Conditions**

Boundary conditions are hard to keep track of. Put the processing for them in one place. Don’t let them leak all over the code.

```
if (level + 1 < tags.length) {
    parts = new Parse(body, tags, level + 1, offset + endTag);
    body = null;
}
```

```
int nextLevel = level + 1;
if (nextLevel < tags.length) {
    parts = new Parse(body, tags, nextLevel, offset + endTag);
    body = null;
}
```

**G34: Functions Should Descend Only One Level of Abstraction**

The statements within a function should all be written at the same level of abstraction, which should be one level below the operation described by the name of the function.

```java
public String render() throws Exception {
    StringBuffer html = new StringBuffer("<hr");
    if (size > 0)
        html.append(" size=\"").append(size + 1).append("\"");
    html.append(">");

    return html.toString();
}
```

This method is mixing at least two levels of abstraction:
1. the notion that a horizontal rule has a size.
2. the syntax of the HR tag itself.

```java
public String render() throws Exception {
    HtmlTag hr = new HtmlTag("hr");
    if (extraDashes > 0)
        hr.addAttribute("size", hrSize(extraDashes));
    return hr.html();
}

private String hrSize(int height) {
    int hrSize = height + 1;
    return String.format("%d", hrSize);
}
```

The `render` function simply constructs an HR tag, without having to know anything about the HTML syntax of that tag. The `HtmlTag` module takes care of all the nasty syntax issues.

**G35: Keep Configurable Data at High Levels**

If you have a constant such as a default or configuration value that is known and expected at a high level of abstraction, do not bury it in a low-level function. Expose it as an argument to that low-level function called from the high-level function.

The configuration constants reside at a very high level and are easy to change. They get passed down to the rest of the application. The lower levels of the application do not own the values of these constants.

**G36: Avoid Transitive Navigation**

In general we don’t want a single module to know much about its collaborators.

`a.getB().getC().doSomething();`

__Writing Shy Code__: Make sure that modules know only about their immediate collaborators and do not know the navigation map of the whole system.

Interpose a `Q` between `B` and `C`. You’d have to find every instance of `a.getB().getC()` and convert it to `a.getB().getQ().getC()`. This is how architectures become rigid.

- ~~Too many modules know too much about the architecture.~~
- Our immediate collaborators offer all the services we need.
- Not to roam through the object graph of the system, hunting for the method we want to call.

`myCollaborator.doSomething();`

## Java

**J1: Avoid Long Import Lists by Using Wildcards**

If you use two or more classes from a package, then import the whole package with
```
import package.*;
```

**J2: Don't Inherit Constants**

This is a hideous practice! The constants are hidden at the top of the inheritance hierarchy. Don’t use inheritance as a way to cheat the scoping rules of the language. Use a static import instead.

Take a look at the following code:
```java
public class HourlyEmployee extends Employee {
    private int tenthsWorked;
    private double hourlyRate;

    public Money calculatePay() {
    int straightTime = Math.min(tenthsWorked, TENTHS_PER_WEEK);
    int overTime = tenthsWorked - straightTime;
    
    return new Money(
            hourlyRate * (tenthsWorked + OVERTIME_RATE * overTime) );
    }
    ... 
}
```
Where did the constants `TENTHS_PER_WEEK` and `OVERTIME_RATE` come from? They might have come from class Employee; so let’s take a look at that:
```java
public abstract class Employee implements PayrollConstants { 
    public abstract boolean isPayday();
    public abstract Money calculatePay();
    public abstract void deliverPay(Money pay);
}
```
Nope, not there. But then where? Look closely at class Employee. It implements `PayrollConstants`.
```java
public interface PayrollConstants {
    public static final int TENTHS_PER_WEEK = 400;
    public static final double OVERTIME_RATE = 1.5;
}
```
Use a static import instead.
```java
import static PayrollConstants.*;
public class HourlyEmployee extends Employee {
    private int tenthsWorked;
    private double hourlyRate;
    public Money calculatePay() {
        int straightTime = Math.min(tenthsWorked, TENTHS_PER_WEEK); int overTime = tenthsWorked - straightTime;

        return new Money(
            hourlyRate * (tenthsWorked + OVERTIME_RATE * overTime) );
    }
    ...
}
```

**J3: Constants versus Enums**

Now that enums have been added to the language (Java 5), use them!

Don’t keep using the old trick of public static final ints.The meaning of ints can get lost. The meaning of enums cannot, because they belong to an enumeration that is named.

What’s more, study the syntax for enums carefully. They can have methods and fields. This makes them very powerful tools that allow much more expression and flexibility than ints.


## Names

**N1: Choose Descriptive Names**

Don’t be too quick to choose a name. Make sure the name is descriptive. Remember that meanings tend to drift as software evolves, so frequently reevaluate the appropriateness of the names you choose.
Names are too important to treat carelessly.

Consider the code below. What does it do? If I show you the code with well-chosen names, it will make perfect sense to you, but like this it’s just a hodge-podge of symbols and magic numbers.
```java
public int x() {
    int q = 0;
    int z = 0;
    for (int kk = 0; kk < 10; kk++) {
        if (l[z] == 10) {
            q += 10 + (l[z + 1] + l[z + 2]);
            z += 1;
        }
    else if (l[z] + l[z + 1] == 10) {
        q += 10 + l[z + 2];
        z += 2;
    } else {
        q += l[z] + l[z + 1];
        z += 2; }
    }
    return q;
}
```
Here is the code the way it should be written. This snippet is actually less complete than the one above. Yet you can infer immediately what it is trying to do, and you could very likely write the missing functions based on that inferred meaning. The magic numbers are no longer magic, and the structure of the algorithm is compellingly descriptive.
```java
public int score() {
    int score = 0;
    int frame = 0;
    for (int frameNumber = 0; frameNumber < 10; frameNumber++) {
        if (isStrike(frame)) {
            score += 10 + nextTwoBallsForStrike(frame);
            frame += 1;
        } else if (isSpare(frame)) {
            score += 10 + nextBallForSpare(frame);
            frame += 2;
        } else {
            score += twoBallsInFrame(frame);
            frame += 2;
        }
    }
    return score;
}
```
The power of carefully chosen names is that they overload the structure of the code with description. That overloading sets the readers’ expectations about what the other functions in the module do. You can infer the implementation of isStrike() by looking at the code above. When you read the isStrike method, it will be “pretty much what you expected.”
```java
private boolean isStrike(int frame) {
    return rolls[frame] == 10;
}
```

**N2: Choose Names at the Appropriate Level of Abstraction**

Don’t pick names that communicate implementation; choose names that reflect the level of abstraction of the class or function you are working in.
 
Consider the `Modem` interface below:
```java
public interface Modem {
    boolean dial(String phoneNumber);
    boolean disconnect();
    boolean send(char c);
    char recv();
    String getConnectedPhoneNumber();
}
```
At first this looks fine. The functions all seem appropriate. Indeed, for many applications they are. But now consider an application in which some modems aren’t connected by dialling. Rather they are connected permanently by hard wiring them together (think of the cable modems that provide Internet access to most homes nowadays). Perhaps some are connected by sending a port number to a switch over a USB connection. Clearly the notion of phone numbers is at the wrong level of abstraction. A better naming strategy for this scenario might be:

```java
public interface Modem {
    boolean connect(String connectionLocator);
    boolean disconnect();
    boolean send(char c);
    char recv();
    String getConnectedLocator();
}
```
Now the names don’t make any commitments about phone numbers. They can still be used for phone numbers, or they could be used for any other kind of connection strategy.

**N3: Use Standard Nomenclature Where Possible**

Names are easier to understand if they are based on existing convention or usage.

For example, if you are using the `DECORATOR` pattern, you should use the word `Decorator` in the names of the decorating classes. For example, `AutoHangupModemDecorator` might be the name of a class that decorates a Modem with the ability to automatically hang up at the end of a session.

Patterns are just one kind of standard. In Java, for example, functions that convert objects to string representations are often named toString.
 
It is better to follow conventions like these than to invent your own.

Teams will often invent their own standard system of names for a particular project.

In short, the more you can use names that are overloaded with special meanings that are relevant to your project, the easier it will be for readers to know what your code is talking about.

**N4: Unambiguous Names**

Choose names that make the workings of a function or variable unambiguous. Consider this example from FitNesse:
```java
private String doRename() throws Exception {
    if(refactorReferences)
        renameReferences();
    renamePage();
    
    pathToRename.removeNameFromEnd();
    pathToRename.addNameToEnd(newName);
    
    return PathParser.render(pathToRename);
}
```
The name of this function does not say what the function does except in broad and vague terms. This is emphasized by the fact that there is a function named renamePage inside the function named doRename! What do the names tell you about the difference between the two functions? Nothing.

A better name for that function is renamePageAndOptionallyAllReferences. This may seem long, and it is, but it’s only called from one place in the module, so it’s explanatory value outweighs the length.

**N5: Use Long Names for Long Scopes**

The length of a name should be related to the length of the scope. You can use very short variable names for tiny scopes, but for big scopes you should use longer names.

Variable names like `i` and `j` are just fine if their scope is five lines long. Consider this snippet from the old standard “Bowling Game”:
```java
private void rollMany(int n, int pins) {
    for (int i=0; i<n; i++)
     g.roll(pins);
}
```
This is perfectly clear and would be obfuscated if the variable `i` were replaced with something annoying like `rollCount`. On the other hand, variables and functions with short names lose their meaning over long distances.

So the longer the scope of the name, the longer and more precise the name should be.

**N6: Avoid Encodings**

Names should not be encoded with type or scope information. Prefixes such as `m_`or `f` are useless in today’s environments. Also project and/or subsystem encodings such as `vis_` (for visual imaging system) are distracting and redundant.
 
Again, today’s environments provide all that information without having to mangle the names. Keep your names free of Hungarian pollution.

**N7: Names Should Describe Side-Effects**

Names should describe everything that a function, variable, or class is or does. Don’t hide side effects with a name. Don’t use a simple verb to describe a function that does more than just that simple action. For example, consider this code from TestNG:
```java
public ObjectOutputStream getOos() throws IOException {
    if (m_oos == null) {
        m_oos = new ObjectOutputStream(m_socket.getOutputStream());
    }
    
    return m_oos;
}
```
This function does a bit more than get an “oos”; it creates the “oos” if it hasn’t been created already. Thus, a better name might be `createOrReturnOos`.

## Tests

**T1: Insufficient Tests**

A test suite should test everything that could possibly break.
 
The tests are insufficient so long as there are conditions that have not been explored by the tests or calculations that have not been validated.

**T2: Use a Coverage Tool!**

Coverage tools reports gaps in your testing strategy. They make it easy to find modules, classes, and functions that are insufficiently tested.
 
Most IDEs give you a visual indication, marking lines that are covered in green and those that are uncovered in red. This makes it quick and easy to find if or catch statements whose bodies haven’t been checked.

**T3: Dont' Skip Trivial Tests**

They are easy to write and their documentary value is higher than the cost to produce them.

**T4: An Ignored Test Is a Question about an Ambiguity**

Sometimes we are uncertain about a behavioral detail because the requirements are unclear. We can express our question about the requirements as a test that is commented out, or as a test that annotated with @Ignore. Which you choose depends upon whether the ambiguity is about something that would compile or not.

**T5: Test Boundary Conditions**

Take special care to test boundary conditions. We often get the middle of an algorithm right but misjudge the boundaries.

**T6: Exhaustively Test Near Bugs**

Bugs tend to congregate. When you find a bug in a function, it is wise to do an exhaustive test of that function. You’ll probably find that the bug was not alone.

**T7:  Patterns of Failure Are Revealing**

Sometimes you can diagnose a problem by finding patterns in the way the test cases fail. This is another argument for making the test cases as complete as possible.
 
Complete test cases, ordered in a reasonable way, expose patterns.

As a simple example, suppose you noticed that all tests with an input larger than five characters failed? Or what if any test that passed a negative number into the second argument of a function failed? Sometimes just seeing the pattern of red and green on the test report is enough to spark the “Aha!” that leads to the solution.

**T8:  Test Coverage Patterns Can Be Revealing**

Looking at the code that is or is not executed by the passing tests gives clues to why the failing tests fail.

**T9: Tests Should Be Fast**

A slow test is a test that won’t get run. When things get tight, it’s the slow tests that will be dropped from the suite. So do what you must to keep your tests fast.

## Conclusion

This list of heuristics and smells could hardly be said to be complete. Indeed, I’m not sure that such a list can ever be complete. But perhaps completeness should not be the goal, because what this list does do is imply a value system.

Indeed, that value system has been the goal, and the topic, of this book. Clean code is not written by following a set of rules. You don’t become a software craftsman by learning a list of heuristics. Professionalism and craftsmanship come from values that drive disciplines.
