# Chapter 8 - Boundaries

Have we used open source or other teams components or subsystems before?
In this chapter, we will look at practices and techniques to keep the boundaries of our software clean when integrating with foreign code.

## Using Third-Party Code

Example: java.util.Map

List of interfaces:
- clear() void - Map
- containsKey(Object key) boolean - Map
- contacinsValue(object value) boolean - Map
- entrySet() Set - Map
- equals(Object o) Object - Map
- get(Object key) Object - Map
- ...

For instance, our application might build up a `Map` and pass it around.
Our intention might be that none of the recipients of our `Map` delete anything in the map.
But at the top of the list is the `clear()` method. Any user of the `Map` can clear it.
`Map` do not reliably constrain the types of objects placed within them. 
Any determined user can add items of any type to any `Map`.

```java
/* Example: Map of Sensors */
Map sensors = new HashMap();
// When some part of the code need to access sensor
Sensor s = (Sensor) sensors.get(sensorId);
```

We will see it over and over again throughout the code. 
This will work, but it's not clean code.

```java
/* Example: Improved by using generics */
Map<Sensor> sensors = new HashMap<Sensor>();
Sensor s = sensors.get(sensorId);
```

However, this doesn't solve the problem that `Map<Sensor>` provides more capability than we need or want.

Passing an instance of `Map<Sensor>` liberally around the system means that there will be a lot of places to fix if the interface `Map` ever changes.

```java
/* Example: Cleaner way to use Map */
public class Sensors {
    private Map sensors = new HashMap();
    
    public Sensor getById(String id) {
        return (Sensor) sensors.get(id);
    }
}
```

The interface at the boundary(`Map`) is hidden. It is able to evolve with very little impact on the rest of the application.
The use of generics is no longer a big issue because the casting and type management is handled inside the `Sensor` class.

We are not suggesting that every use of `Map` be encapsulated in this form. 
Rather, we are advising not to pass `Map`s (or any other interface at a boundary) around our system.
If we use a boundary interface like `Map`, keep it inside the class, or close family of classes, where it is used.
Avoid returning it from, or accepting it as an argument to, public APIs.

## Exploring and Learning Boundaries

Instead of experimenting and trying out new stuff in our production code, 
we could write some tests to explore our understanding of the third-party code.
Jim Newkirk called such tests **learning tests**.

In learning tests, we call the third-party API, as we expect to use it in our application.
We're essentially doing controlled experiments that check our understanding of that API.
The tests focus on what we want out of the API.

## Learning `log4j`

Write our first test case, expecting it to write "hello" to the console.

```java
@Test
public void testLogCreate() {
    Logger logger = Logger.getLogger("MyLogger");
    logger.info("hello");
}
```

When we run it, the logger produces an error that tells us we need something called an `Appender`. 
After a little more reading, we find that there is a `ConsoleAppender`.

```java
@Test
public void testLogAddAppender() {
    Logger logger = Logger.getLogger("MyLogger");
    ConsoleAppender appender = new ConsoleAppender();
    logger.addAppender(appender);
    logger.info("hello");
}
```

This time we find that the `Appender` has no output stream.

```java
@Test
public void testLogAddAppender() {
    Logger logger = Logger.getLogger("MyLogger");
    logger.removeAllAppenders();
    logger.addAppender(new ConsoleAppender(
            new PatternLayout("%p %t %m%n"),
            ConsoleAppender.SYSTEM_OUT));
    logger.info("hello");
}
```

That worked; a log message that includes "hello" came out on the console!
It seems odd that we have to tell the `ConsoleAppender` that it writes to the console.

Interestingly enough, when we remove the `ConsoleAppender.SYSTEM_OUT` argument, we see that "hello" is still printed.
But when we take out the `PatternLayout`, it once again complains about the lack of an output stream.
This is very strange behavior.

The default `ConsoleAppender` constructor is "unconfigured". This cause inconsistency.

After some readings and testings, we've encoded that knowledge into a set of simple unit tests.

```java
public class LogTest {
    private Logger logger;
    
    @Before
    public void initialize() {
        logger = Logger.getLogger("logger");
        logger.removeAllAppenders();
        Logger.getRootLogger().removeAllAppenders();
    }
    
    @Test
    public void basicLogger() {
        BasicConfigurator.configure();
        logger.info("basicLogger");
    }
    
    @Test
    public void addAppenderWithStream() {
        logger.addAppender(new ConsoleAppender(
                new PatternLayout("%p %t %m%n"),
                ConsoleAppender.SYSTEM_OUT));
        logger.info("addAppenderWithStream");
    }
    
    @Test
    public void addAppenderWithoutStream() {
        logger.addAppenedr(new ConsoleAppender(
                new PatternLayout("%p %t %m%n")));
        logger.info("addAppenderWithoutStream");
    }
}
```

Now we know how to get a simple console logger initialized, 
and we can encapsulate that knowledge into our own logger class so that the rest of our application is isolated from the `log4j` boundary interface.

## Learning Tests Are Better Than Free

The learning tests were precise experiments that helped increase our understanding.

When there are new releases of the third-party packages, we run the learning tests to see whether there are behavioral differences.

If the third-party package changes in some way incompatible without tests, we will find out right away.

A clean boundary should be supported by a set of outbound tests that exercise the interface the same way the production does.
Without these boundary tests to ease the migration, we might be tempted to stay with the old version longer than we should.

## Using Code That Does Not Yet Exist

There is another kind of boundary, one that separates the known from the unknown.

For example when we are collaborating our project with another team, 
on some occasion the other team had not gotten to the point of defining their interface. 
We did not want to be blocked, so we started our work far away from the unknown part of the code.

Firstly, we define our own interface called `Transmitter`. We gave it a method called `transmit`.
This was the interface we wished we had.
One good thing about writing the interface we wish we had is that it's under our control.

![Transmitter Adapter](https://ewegithub.sb.karmalab.net/EWE/clean-code-series/blob/master/images/chapter-08-figure-01.png "Transmitter Adapter")

We insulted the `CommunicationController` class from the Transmitter API (which was out of our control and undefined).
By using our own application-specific interface, we kept our `CommunicationController` code clean and expressive.
Once the Transmitter API was defined, we wrote the `TransmitterAdapter` to bridge the gap.
The *Adapter* encapsulated the interaction with the API and provides a single place to change when the API evolves.

This design also gives us a very convenient seam in the code for testing.
Using a suitable `FakeTransmitter`, we can test the `CommunicationController` class.
We can also create boundary tests once we have the `TransmitterAPI` that make sure we are using the API correctly.

## Clean Boundaries

Good software designs accommodate change without huge investments and rework.
When we use code that is out of our control, special care must be taken to protect our investment and make sure future changes is not too costly.

The code at the boundaries needs clear separation and tests that define expectations.
We should avoid letting too much of our code know about the third-party particulars. 
It's better to depend on something we can control than on something that we don't control.

This promotes internal consistent usage across the boundary and has fewer maintenance points when the third-party code changes.
