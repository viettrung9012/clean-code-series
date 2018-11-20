# Chapter 5 - Formatting
##Purpose
Code formatting is about communication, and it is professional developer’s first order of business.

The functionality that you create today has a good chance of changing in the next release, but the readability of your code will have a profound effect on all the changes that will ever be made.

Code keeps changing, it’s the coding style and discipline that survive.

## Vertical Formatting: 
Nearly all code is read left to right and top to bottom. Each line represents an expression or a clause, and each group of lines represents a complete thought. Those thoughts should be separated from each other with blank lines.

Example:

```java
package fitnesse.wikitext.widgets;
import java.util.regex.*;
public class BoldWidget extends ParentWidget {
    public static final String REGEXP = "'''.+?'''";
    private static final Pattern pattern = Pattern.compile("'''(.+?)'''",
        Pattern.MULTILINE + Pattern.DOTALL);

    public BoldWidget(ParentWidget parent, String text) throws Exception {
        super(parent);
        Matcher match = pattern.matcher(text);
        match.find();
        addChildWidgets(match.group(1));
    }

    public String render() throws Exception {
        StringBuffer html = new StringBuffer("<b>");
        html.append(childHtml()).append("</b>");
        return html.toString();
    }
}
```

Without blank lines:

```java

package fitnesse.wikitext.widgets;
import java.util.regex.*;
public class BoldWidget extends ParentWidget {
    public static final String REGEXP = "'''.+?'''";
    private static final Pattern pattern = Pattern.compile("'''(.+?)'''",
        Pattern.MULTILINE + Pattern.DOTALL);
    public BoldWidget(ParentWidget parent, String text) throws Exception {
        super(parent);
        Matcher match = pattern.matcher(text);
        match.find();
        addChildWidgets(match.group(1));
    }
    public String render() throws Exception {
        StringBuffer html = new StringBuffer("<b>");
        html.append(childHtml()).append("</b>");
        return html.toString();
    }
}
```

## Vertical Density
Lines of code that are tightly related should appear vertically dense

Example:

```java
public class ReporterConfig {
    /**
     * The class name of the reporter listener 
    */
    private String m_className;

    /**
     * The properties of the reporter listener 
    */
    private List < Property > m_properties = new ArrayList < Property > ();

    public void addProperty(Property property) {
        m_properties.add(property);
    }
}
```

Removing useless comments:

```java
public class ReporterConfig {
    private String m_className;
    private List < Property > m_properties = new ArrayList < Property > ();

    public void addProperty(Property property) {
        m_properties.add(property);
    }
}
```

## Variable Declarations:
Variables should be declared as close to their usage as possible.

Example: 

```java
public int countTestCases() {
    int count = 0;
    for (Test each: tests)
        count += each.countTestCases();
    return count;
}
```
In rare cases a variable might be declared at the top of a block or just before a loop in a long-ish function.

Example:

```java
… 
for (XmlTest test: m_suite.getTests()) {
    TestRunner tr = m_runnerFactory.newTestRunner(this, test);
    tr.addListener(m_textReporter);
    m_testRunners.add(tr);
    invoker = tr.getInvoker();
    for (ITestNGMethod m: tr.getBeforeSuiteMethods()) {
        beforeSuiteMethods.put(m.getMethod(), m);
    }
    for (ITestNGMethod m: tr.getAfterSuiteMethods()) {
        afterSuiteMethods.put(m.getMethod(), m);
    }
}
… 
```

Instance variable, on the other hand, should be declared at the top of the class. This should not increase the vertical distance of these variables, because in a well-designed class, they are used by many, if not all, of the methods of the class.

Example: 
 
Quite hard to find out the variables declared in the below example, they are hidden in middle of method implementations:

```java

public class TestSuite implements Test {
    static public Test createTest(Class << ? extends TestCase > theClass, String name) {
        ...
    }
    
    public static Constructor << ? extends TestCase > getTestConstructor(Class << ? extends TestCase > theClass) throws NoSuchMethodException {
        ...
    }
    
    public static Test warning(final String message) { 
        ...
    }
    
    private static String exceptionToString(Throwable t) { 
        ...
    }
    
    private String fName;
    private Vector < Test > fTests = new Vector < Test > (10);
    
    public TestSuite() {}
    
    public TestSuite(final Class << ? extends TestCase > theClass) { 
        ...
    }
    
    public TestSuite(Class << ? extends TestCase > theClass, String name) { 
        ...
    }
    ...............
}
```

Dependent Functions, If one function calls another, they should be vertically close, and the caller should be above the callee, if at all possible. 

Example:

```java
public class WikiPageResponder implements SecureResponder {
    protected WikiPage page;
    protected PageData pageData;
    protected String pageTitle;
    protected Request request;
    protected PageCrawler crawler;
 
   public Response makeResponse(FitNesseContext context, Request request) throws Exception {
        String pageName = getPageNameOrDefault(request, "FrontPage");
        loadPage(pageName, context);
        if (page == null)
            return notFoundResponse(context, request);
        else
            return makePageResponse(context);
    }

    private String getPageNameOrDefault(Request request, String defaultPageName) {
        String pageName = request.getResource();
        if (StringUtil.isBlank(pageName))
            pageName = defaultPageName;
        return pageName;
    }

    protected void loadPage(String resource, FitNesseContext context) throws Exception {
        WikiPagePath path = PathParser.parse(resource);
        crawler = context.root.getPageCrawler();
        crawler.setDeadEndStrategy(new VirtualEnabledPageCrawler());
        page = crawler.getPage(context.root, path);
        if (page != null)
            pageData = page.getData();
    }

    private Response notFoundResponse(FitNesseContext context, Request request) throws Exception {
        return new NotFoundResponder().makeResponse(context, request);
    }

    private SimpleResponse makePageResponse(FitNesseContext context) throws Exception {
        pageTitle = PathParser.render(crawler.getFullPath(page));
        String html = makeHtml(context);
        SimpleResponse response = new SimpleResponse();
        response.setMaxAge(0);
        response.setContent(html);
        return response;
    }
...
```    

Conceptual Affinity, Certain bits of code want to be near other bits. They have a certain conceptual affinity. The stronger that affinity, the less vertical distance there should be between them.

This affinity might be based on a direct dependence, such as one function calling another, or a function using a variable. But there are other possible causes of affinity. Affinity might be caused because a group of functions perform a similar operation.

Example: 
```java

public class Assert {
    static public void assertTrue(String message, boolean condition) {
        if (!condition) 
            fail(message);
    }
    static public void assertTrue(boolean condition) {
        assertTrue(null, condition);
    }
    static public void assertFalse(String message, boolean condition) {
        assertTrue(message, !condition);
    }
    static public void assertFalse(boolean condition) {
        assertFalse(null, condition);
    }
    ...
}
```
These functions have a strong conceptual affinity because they share a common naming scheme and perform variations of the same basic task. The fact that they call each other is secondary. Even if they didn’t, they would still want to be close together.

## Vertical Ordering:

In general we want function call dependencies to point in the downward direction. That is, a function that is called should be below a function that does the calling.2 This creates a nice flow down the source code module from high level to low level.

Example: Newspaper articles

## Horizontal Formatting:

One should strive to keep our lines short. Author’s recommendation is 120 characters

## Horizontal Openness and Density:

Use horizontal white space to associate things that are strongly related and disassociate things that are more weakly related

Example:

```java
private void measureLine(String line) {
    lineCount++;
    int lineSize = line.length();
    totalChars += lineSize;
    lineWidthHistogram.addLine(lineSize, lineCount);
    recordWidestLine(lineSize);
}

```

Another use of white space is to accentuate the precedence of operators

(-b + Math.sqrt(determinant)) / (2*a);

## Horizontal Alignment:

Below alignment seems to emphasize the wrong things and lead eye away from true intent.

```java
private Socket        socket; 
private InputStream   input; 
private OutputStream  output;
```

Instead keeping variable unaligned points out any important deficiency. i.e., the long list of variables. Class may need a split up.

```java
private Socket socket; 
private InputStream input; 
private OutputStream output;
```

## Indentation:

Program readability heavily relies on Indentations. Visually lined up lines on the left helps to see what scope they appear in. This doesn’t limits not just for if or while statements. new method declarations, new variables, and even new classes are scanned for alignment.

Without indentation, programs would be virtually unreadable by humans:

##UnIndented Program: 

```java
public class FitNesseServer implements SocketServer { private FitNesseContext context; public FitNesseServer(FitNesseContext context) { this.context = context; } public void serve(Socket s) { serve(s, 10000); } public void serve(Socket s, long requestTimeout) { try { FitNesseExpediter sender = new FitNesseExpediter(s, context); sender.setRequestParsingTimeLimit(requestTimeout); sender.start(); } catch(Exception e) { e.printStackTrace(); } } }
```

## Indented Program:
```java
public class FitNesseServer implements SocketServer {
    private FitNesseContext context;

    public FitNesseServer(FitNesseContext context) {
        this.context = context;
    }

    public void serve(Socket s) {
        serve(s, 10000);
    }

    public void serve(Socket s, long requestTimeout) {
        try {
            FitNesseExpediter sender = new FitNesseExpediter(s, context);
            sender.setRequestParsingTimeLimit(requestTimeout);
            sender.start();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

## Breaking Indentation - Dummy Scopes:

Sometimes the body of a while or for statement is a dummy, as shown below. Unless you make that semicolon visible by indenting it on it’s own line, it’s just too hard to see.

```java
    while (dis.read(buf, 0, readBufferSize) != -1) 
        ;
```
