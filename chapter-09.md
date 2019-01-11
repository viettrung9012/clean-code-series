# Chapter 9 - Unit Tests

## Unit Tests Evolution:

"That was my test! Once I saw it work and demonstrated it to my colleagues, I threw the test code away".

From above statement, we’ve come a long way. Nowadays I would write a test that made sure that every part of that code worked as I expected it to. 

I would isolate my code from the operating system, rather than just calling the standard timing functions, I would mock out those timing functions so that I had absolute control over the time. 

I would make sure that tests are convenient to run for anyone else who needed to work with the code.

I would ensure that the tests and the code were checked in together into the same source package.

Yes, we’ve come a long way; but we have farther to go. The Agile and TDD movements have encouraged many programmers to write automated unit tests. 

But in the mad rush to add testing to our discipline, many programmers have missed some of the important points of writing good tests.


## The Three Laws of TDD: 

By now everyone knows that TDD asks us to write unit tests first, before we write production code. 
Consider the following three laws:

First Law - You may not write production code until you have written a failing unit test.

Second Law - You may not write more of a unit test than is sufficient to fail, and not compiling
is failing.

Third Law - You may not write more production code than is sufficient to pass the currently
failing test.


# Keeping Tests Clean:

These three laws lock you into a cycle that is perhaps thirty seconds long. The tests and the production code are written together, with the tests just a few seconds ahead of the production code.

If we work this way, we will write dozens of tests every day, hundreds of tests every month, and thousands of tests every year. 
The tests will then cover virtually all of our production code.


# Tests Enable the -ilities:

If you don’t keep your tests clean, you will lose them. And without them, your production code will not be flexible. Yes, you read that correctly. It is unit tests that keep our code flexible, maintainable, and reusable. 

The reason is simple. If you have tests, you do not fear making changes to the code! Without tests every change is a possible
bug. Without tests you will be reluctant to make changes because of the fear that you will introduce undetected bugs.

So if your tests are dirty, then your ability to change your code is hampered, and you begin to lose the ability to improve the structure of that code. The dirtier your tests, the dirtier your code becomes. Eventually you lose the tests, and your code rots.

In the world of software architecture there are many “-ilities” you must take into consideration with every project. 
Here is a quick list outlining few important “-ilities” :

1. Usability - How effectively end users can use, learn, or control the system.

2. Maintainability ( or Flexibility / Testability) - How brittle the code is to change.

3. Scalability - Ability for your program to gracefully meet the demand of stress caused by increased usage. 
In short, ensuring your program doesn’t slow or bust when pounded by more users than you originally anticipated.

4. Availability (or Reliability) - How long the system is up and running and the Mean Time Between Failure (MTBF) is known as the availability of a program.

5. Extensibility - Are there points in the system where changes can be made with (or without) program changes?

Can the database schema flex to accommodate change?
Does the system allow Inversion of Control (IoC)?
Can end users extend the system (scripts, user defined fields, etc)?
Can 3rd party developers leverage your system?

6. Portability - Portability is the ability for your application to run on numerous platforms.
Can the data be migrated to other systems?
For web applications, which browsers does your web app support?
Which operating systems does your program run on?


## Clean Tests

What makes a clean test? Three things. Readability, readability, and readability. 
Readability is perhaps even more important in unit tests than it is in production code.

Clarity, simplicity and density of expression makes tests readable. In a test you should say a lot with as few expressions as possible.

Consider the code below in Listing 9-1. These three tests are difficult to understand and can certainly be improved. First, there is a terrible amount of duplicate code in the repeated calls to addPage and assertSubString. 
This code is just loaded with details that interfere with the expressiveness of the test.

Listing 9-1
SerializedPageResponderTest.java

```java
public void testGetPageHieratchyAsXml() throws Exception {
 crawler.addPage(root, PathParser.parse("PageOne"));
 crawler.addPage(root, PathParser.parse("PageOne.ChildOne"));
 crawler.addPage(root, PathParser.parse("PageTwo"));
 request.setResource("root");
 request.addInput("type", "pages");
 Responder responder = new SerializedPageResponder();
 SimpleResponse response =
  (SimpleResponse) responder.makeResponse(
   new FitNesseContext(root), request);
 String xml = response.getContent();
 assertEquals("text/xml", response.getContentType());
 assertSubString("<name>PageOne</name>", xml);
 assertSubString("<name>PageTwo</name>", xml);
 assertSubString("<name>ChildOne</name>", xml);
}

public void testGetPageHieratchyAsXmlDoesntContainSymbolicLinks()
throws Exception {
 WikiPage pageOne = crawler.addPage(root, PathParser.parse("PageOne"));
 crawler.addPage(root, PathParser.parse("PageOne.ChildOne"));
 crawler.addPage(root, PathParser.parse("PageTwo"));
 PageData data = pageOne.getData();
 WikiPageProperties properties = data.getProperties();
 WikiPageProperty symLinks = properties.set(SymbolicPage.PROPERTY_NAME);
 symLinks.set("SymPage", "PageTwo");
 pageOne.commit(data);
 request.setResource("root");
 request.addInput("type", "pages");
 Responder responder = new SerializedPageResponder();
 SimpleResponse response =
  (SimpleResponse) responder.makeResponse(
   new FitNesseContext(root), request);
 String xml = response.getContent();
 assertEquals("text/xml", response.getContentType());
 assertSubString("<name>PageOne</name>", xml);
 assertSubString("<name>PageTwo</name>", xml);
 assertSubString("<name>ChildOne</name>", xml);
 assertNotSubString("SymPage", xml);
}

public void testGetDataAsHtml() throws Exception {
 crawler.addPage(root, PathParser.parse("TestPageOne"), "test page");
 request.setResource("TestPageOne");
 request.addInput("type", "data");
 Responder responder = new SerializedPageResponder();
 SimpleResponse response =
  (SimpleResponse) responder.makeResponse(
   new FitNesseContext(root), request);
 String xml = response.getContent();
 assertEquals("text/xml", response.getContentType());
 assertSubString("test page", xml);
 assertSubString("<Test", xml);
}
```

The PathParser calls, transform strings into PagePath instances used by the crawlers. This transformation is completely irrelevant to the test at hand and serves only to complicate the intent. 

The details surrounding the creation of the responder and the gathering and casting of the response are also just noise. Then there’s the clumsy way that the request URL is built from a resource and an argument.

In the end, this code was not designed to be read. The poor reader is overwhelm with a swarm of details that must be understood before the tests make any real sense.

Now consider the improved tests in Listing 9-2. These tests do the exact same thing,but they have been refactored into a much cleaner and more explanatory form.


Listing 9-2
SerializedPageResponderTest.java (refactored)

```java
public void testGetPageHierarchyAsXml() throws Exception {
 makePages("PageOne", "PageOne.ChildOne", "PageTwo");
 submitRequest("root", "type:pages");
 assertResponseIsXML();
 assertResponseContains(
  "<name>PageOne</name>", "<name>PageTwo</name>", "<name>ChildOne</name>"
 );
}

public void testSymbolicLinksAreNotInXmlPageHierarchy() throws Exception {
 WikiPage page = makePage("PageOne");
 makePages("PageOne.ChildOne", "PageTwo");
 addLinkTo(page, "PageTwo", "SymPage");
 submitRequest("root", "type:pages");
 assertResponseIsXML();
 assertResponseContains(
  "<name>PageOne</name>", "<name>PageTwo</name>", "<name>ChildOne</name>"
 );
 assertResponseDoesNotContain("SymPage");
}

public void testGetDataAsXml() throws Exception {
 makePageWithContent("TestPageOne", "test page");
 submitRequest("TestPageOne", "type:data");
 assertResponseIsXML();
 assertResponseContains("test page", "<Test");
}
```


## BUILD-OPERATE-CHECK pattern:

Tests should be clearly split into three parts. The first part builds up the test data, the second part operates on that test data, and the third part checks that the operation yielded the expected results.

Majority of annoying detail should be eliminated, the tests should get right to the point and use only the data types and functions that they truly need. Anyone who reads these tests should be able to work out what they do very quickly, without being misled or overwhelmed by details.

Build Operate Check pattern is a generalization of the Given/When/Then style. where the four phases include:
1. SetUp or Build.

2. Execute or Operate.

3. Verify or Check.

4. TearDown (optional).


## Domain-Specific Testing Language

Always build a domain-specific language for your tests (Listing 9-2), rather than using the APIs that programmers use to manipulate the system, we build up a set of functions and utilities that make use of those APIs and that make the tests more convenient to write and easier to read. 
Domain-specific language are a testing language that programmers use to help themselves to write their tests and to help those who must read those tests later on.


## A Dual Standard
The code within the testing API does have a different set of engineering standards than production code. It must still be simple, concise and expressive, but it need not be as efficient as production code. 
After all, it runs in a test environment, not a production environment, and those two environment have very different needs.

Consider the 'environment control system' test in Listing 9-3. Without going into the details you can tell that this test checks that the low temperature alarm, the heater, and the blower are all turned on when the temperature is “way too cold.”

Listing 9-3
EnvironmentControllerTest.java

```java
@Test
public void turnOnLoTempAlarmAtThreashold() throws Exception {
hw.setTemp(WAY_TOO_COLD);
controller.tic();
assertTrue(hw.heaterState());
assertTrue(hw.blowerState());
assertFalse(hw.coolerState());
assertFalse(hw.hiTempAlarm());
assertTrue(hw.loTempAlarm());
}
```

There are lots of details here. For example, what is that tic function all about? In fact, I’d rather want you not to worry about that while reading this test. I’d rather want you just worry about whether you agree that the end state of the system is consistent with the temperature being “way too cold.”

Notice, as you read the test, that your eye needs to bounce back and forth between the name of the state being checked, and the sense of the state being checked. It makes the test hard to read.

I improved the reading of this test greatly by transforming it into Listing 9-4.

Listing 9-4
EnvironmentControllerTest.java (refactored)

```java
@Test
public void turnOnLoTempAlarmAtThreshold() throws Exception {
wayTooCold();
assertEquals("HBchL", hw.getState());
}
```

Of course I hid the detail of the tic function by creating a wayTooCold function. But the thing to note is the strange string in the assertEquals. Upper case means “on,” lower case means “off,” and the letters are always in the following order: {heater, blower, cooler, hi-temp-alarm, lo-temp-alarm}.

Even though this is close to a violation of the rule about mental mapping, it seems appropriate in this case. Notice, once you know the meaning, your eyes glide across that string and you can quickly interpret the results. 
Reading the test becomes almost a pleasure. Just take a look at Listing 9-5 and see how easy it is to understand these tests.

Listing 9-5
EnvironmentControllerTest.java (bigger selection)

```java
@Test
public void turnOnCoolerAndBlowerIfTooHot() throws Exception {
 tooHot();
 assertEquals("hBChl", hw.getState());
}
@Test
public void turnOnHeaterAndBlowerIfTooCold() throws Exception {
 tooCold();
 assertEquals("HBchl", hw.getState());
}
@Test
public void turnOnHiTempAlarmAtThreshold() throws Exception {
 wayTooHot();
 assertEquals("hBCHl", hw.getState());
}
@Test
public void turnOnLoTempAlarmAtThreshold() throws Exception {
 wayTooCold();
 assertEquals("HBchL", hw.getState());
}
```

The getState function is shown in Listing 9-6.

Listing 9-6
MockControlHardware.java

```java
public String getState() {
String state = "";
state += heater ? "H" : "h";
state += blower ? "B" : "b";
state += cooler ? "C" : "c";
state += hiTempAlarm ? "H" : "h";
state += loTempAlarm ? "L" : "l";
return state;
}
```


## One Assert per Test

There is a school of thought that says that every test function in a JUnit test should have one and only one assert statement. This rule may seem harsh, but the advantage can be seen in Listing 9-5. Those tests come to a single conclusion that is quick and easy to understand.

Listing 9-7
SerializedPageResponderTest.java (Single Assert)

```java
public void testGetPageHierarchyAsXml() throws Exception {
 givenPages("PageOne", "PageOne.ChildOne", "PageTwo");
 whenRequestIsIssued("root", "type:pages");
 thenResponseShouldBeXML();
}
public void testGetPageHierarchyHasRightTags() throws Exception {
 givenPages("PageOne", "PageOne.ChildOne", "PageTwo");
 whenRequestIsIssued("root", "type:pages");
 thenResponseShouldContain(
  "<name>PageOne</name>", "<name>PageTwo</name>", "<name>ChildOne</name>"
 );
}
```

Notice that I have changed the names of the functions to use the common given-when-then convention. This makes the tests even easier to read. Unfortunately, splitting the tests as shown results in a lot of duplicate code.

We can eliminate the duplication by using the TEMPLATE METHOD pattern and putting the given/when parts in the base class, and the then parts in different derivatives. Or we could create a completely separate test class and put the given and when parts in the @Before function, and the when parts in each @Test function. 


## Single Concept per Test

Perhaps a better rule is that we want to test a single concept in each test function. We don’t want long test functions that go testing one miscellaneous thing after another. Listing 9-8 is an example of such a test. 
This test should be split up into three independent tests because it tests three independent things.

Listing 9-8

```java
/**
 * Miscellaneous tests for the addMonths() method.
 */
public void testAddMonths() {
 SerialDate d1 = SerialDate.createInstance(31, 5, 2004);
 SerialDate d2 = SerialDate.addMonths(1, d1);
 assertEquals(30, d2.getDayOfMonth());
 assertEquals(6, d2.getMonth());
 assertEquals(2004, d2.getYYYY());
 SerialDate d3 = SerialDate.addMonths(2, d1);
 assertEquals(31, d3.getDayOfMonth());
 assertEquals(7, d3.getMonth());
 assertEquals(2004, d3.getYYYY());
 SerialDate d4 = SerialDate.addMonths(1, SerialDate.addMonths(1, d1));
 assertEquals(30, d4.getDayOfMonth());
 assertEquals(7, d4.getMonth());
 assertEquals(2004, d4.getYYYY());
}
```

The three test functions probably ought to be like this:

• Given the last day of a month with 31 days (like May):
1. When you add one month, such that the last day of that month is the 30th (like June), then the date should be the 30th of that month, not the 31st.

2. When you add two months to that date, such that the final month has 31 days, then the date should be the 31st.


• Given the last day of a month with 30 days in it (like June):
1. When you add one month such that the last day of that month has 31 days, then the date should be the 30th, not the 31st.

Stated like this, you can see that there is a general rule hiding amidst the miscellaneous tests. When you increment the month, the date can be no greater than the last day of the month. This implies that incrementing the month on February 28th should yield March 28th. That test is missing and would be a useful test to write.


## F.I.R.S.T.

Clean tests follow five other rules that form the above acronym:

Fast - Tests should be fast. When tests run slow, you won’t want to run them frequently. If you don’t run them frequently, you won’t find problems early enough to fix them easily. You won’t feel as free to clean up the code. Eventually the code will begin to rot.

Independent -  Tests should not depend on each other. You should be able to run each test independently. When tests depend on each other, diagnosis becomes difficult and it hides downstream defects.

Repeatable - Tests should be repeatable in any environment. If your tests aren’t repeatable in any environment, then you’ll always have an excuse for why they fail. You’ll also find yourself unable to run the tests when the environment isn’t available.

Self-Validating - The tests should have a Boolean output. Either they pass or fail. You should not have to read through a log file to tell whether the tests pass. If the tests aren’t self-validating, then failure can become subjective and running the tests can require a long manual evaluation.

Timely - The tests need to be written in a timely fashion. Unit tests should be written just before the production code that makes them pass. If you write tests after the production code, then you may find the production code to be hard to test.


## Conclusion
- Tests are as important to the health of a project as the production code is. Perhaps they are even more important, because tests preserve and enhance the flexibility, maintainability, and reusability of the production code. 
- Keep your tests constantly clean. Work to make them expressive and succinct. 
- Invent testing APIs that act as domain-specific language that helps you write the tests. 
- If you let the tests rot, then your code will rot too. Keep your tests clean.
