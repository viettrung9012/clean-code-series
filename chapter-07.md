# Chapter 7 - Error Handling

Error handling is one of the things that we have to do to handle things that go wrong.

Error handling is important, but it should not obscure logic.

## Use Exceptions Rather Than Return Codes:

```java
/* Example: Set an error flag returned an error code */
public class DeviceController {
    public void sendShutDown() {
        DeviceHandle handle = getHandle(DEV1);
        // Check the state of the device
        if (handle != DeviceHandle.INVALID) {
            // Save the device status to the record field
            retrieveDeviceRecord(handle);
            // If not suspended, shut down
            if (record.getStatus() != DEVICE_SUSPENDED) {
                pauseDevice(handle);
                clearDeviceWorkQueue(handle);
                closeDevice(handle);
            } else {
                logger.log("Device suspended. Unable to shut down");
            }
        } else {
        logger.log("Invalid handle for: " + DEV1.toString()); }
    }
}
```

The problem with the above approaches is that the caller must check for errors immediately after the call.
Unfortunately, it's easy to forget. For this reason, it is better to throw an exception when you encounter an error.
Its logic is not obscured by error handling.

```java
/* Example: Throw exceptions in method that can detect errors. */
public class DeviceController {
    public void sendShutDown() {
        try {
            tryToShutDown();
        } catch (DeviceShutDownError e) {
            logger.log(e);
        }
    }
    
    private void tryToShutDown() throws DeviceShutDownError {
        DeviceHandle handle = getHandle(DEV1);
        DeviceRecord record = retrieveDeviceRecord(handle);
        
        pauseDevice(handle);
        clearDeviceWorkQueue(handle);
        closeDevice(handle);
    }
    
    private DeviceHandle getHandle(DeviceID id) {
        throw new DeviceShutDownError("Invalid handle for: " + id.toString());
    }
}
```

The code is better because two concerns that were tangled, the algorithm for device shutdown and error handling, are now separated.

## Write `Try-Catch-Finally` Statement First

`try-catch-finally` define a scope within the program.
When executing code in the `try` portion, execution might abort at any time and then resume at the `catch`.

No matter whether an exception occurs in `try` block or not, `finally` will always be executed.

```java
/* Example: try-catch-finally */
public class File {
    public void write(String filename, String fileContent) {
        FileOutputStream out = null;
        try {
            out = new FileOutputStream(filename);
            out.write(fileContent);
        } catch (IOException e) {
            logger.error("Error writing to " + filename + ": " + e.getMessage(), e);
        } finally {
            IOUtils.closeQuietly(out);
        }
    }
}
```

## Use Unchecked Exceptions

We thought that checked exceptions were a great idea; and yes, they can yield some benefit. 
However, they are not necessary for the production of robust software.

Checked exceptions are violating Open/Closed Principle.

If we throw a checked exception from a method in our code and the `catch` is three levels above, 
we must declare that exception in the signature of each method between our code and the catch.
This means that a change at a low level of the software can force signature changes on many higher levels.

Consider the calling of a large system. Functions at the top call functions below them, which call more functions below them, ad infinitum.
Now let's say one of the lowest level functions is modified in such a way that it must throw an exception.
If that exception is checked, then the function signature must add a throws clause. 
But this means that every function that calls our modified function must also be modified either to catch the new exception or to append the appropriate `throws` clause to its signature.
The net result is a cascade of changes that work their way from the lowest levels of the software to the highest!
Encapsulation is broken because all functions in the path of a throw must know about details of that low-level exception.

Checked exceptions can sometimes be useful if we are writing a critical library.

## Provide Context with Exceptions

Each exception that we throw should provide enough context to determine the source and location of an error.
Sometimes stack trace can't tell us the intent of the operation that failed.

Create informative error messages and pass them along with exceptions.
Mention the operation that failed and the type of failure.
If we are logging in our application, pass along enough information to be able to log the error in `catch`.

## Define Exception Classes in Terms of a Caller's Needs

```java
/* Example: Poor exception classification */
ACMEPort port = new ACMEPort(12);
try {
    port.open();
} catch (DeviceResponseException e) {
    reportPortError(e);
    logger.log("Device response exception", e);
} catch (ATM1212UnlockedException e) {
    reportPortError(e);
    logger.log("Unlock exception", e);
} catch (GMXError e) {
    reportPortError(e);
    logger.log("Device response exception");
} finally {
    ...
}
```

The statement contains a lot of duplication. 
In most exception handling situations, the work that we do is relatively standard regardless of the actual cause.
We have to record an error and proceed.

In this case, because we know that the work that we are doing is roughly the same, regardless of the exception, 
we can simplify our code considerably by wrapping the API that we are calling and making sure that it returns a common exception type.

```java
/* Example: Catch similar exception with wrapper */
LocalPort port = new LocalPort(12);
try {
    port.open();
} catch (PortDeviceFailure e) {
    reportError(e);
    logger.log(e.getMessage(), e);
} finally {
    ...
}

public class LocalPort {
    private ACMEPort innerPort;

    public LocalPort(int portNumber) {
        innerPort = new ACMEPort(portNumber);
    }

    public void open() {
        try {
            innerPort.open();
        } catch (DeviceResponseException e) {
            throw new PortDeviceFailure(e);
        } catch (ATM1212UnlockedException e) {
            throw new PortDeviceFailure(e);
        } catch (GMXError e) {
            throw new PortDeviceFailure(e);
        }
    }
}
```

Wrapping third-party APIs is a best practice.
When we wrap a third-party API, we minimize dependencies upon it.
We can choose to move to a different library in the future without much penalty.
Wrapping also makes it easier to mock out third-party calls when testing our own code.

## Define the Normal Flow

Most of the time wrapping external APIs are a great approach, 
but there are sometimes when we may not want to abort.

```java
/* Example: Exception clutters the logic */
try {
    MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
    m_total += expenses.getTotal();
} catch(MealExpensesNotFound e) {
    m_total += getMealPerDiem();
}
```

We can change the `ExpenseReportDAO` so that it always returns a `MealExpense` object. 
If there are not meal expenses, it returns a `MealExpense` object that returns the *per diem* as its total.

```java
MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
m_total += expenses.getTotal();

public class PerDiemMealExpenses implements MealExpenses {
    public int getTotal() {
        // return the per diem default
    }
}
```

## Don't Return Null

```java
/* Example: Method with too many null check */
public void registerItem(Item item) {
    if (item != null) {
        ItemRegistry registry = persistentStore.getItemRegistry();
        if (registry != null) {
            Item existing = registry.getItem(item.getId());
            if (existing.getBillingPeriod().hasRetailOwner()) {
                existing.register(item);
            }
        }
    }
}
```

When we return `null`, we are essentially creating work for ourselves and foisting problems upon our callers.
All it takes is one missing `null` check to send an application spinning out of control.

In the 3rd line, what would have happened at runtime if `persistentStore` were `null`?

`NullPointerException` at runtime. 
Even we catching it at the top level or not, they are still bad.

If we are tempted to return `null` from a method, consider throwing an exception or returning a `SPECIAL CASE` object instead.

If we are calling a `null`-returning method from a third-party API, 
consider wrapping that method with a method that either throws an exception or return a special case object.

```java
/* Example: getEmployees can return null */
List<Employee> employees = getEmployees();
if (employees != null) {
    for (Employee e : employees) {
        totalPay += e.getPay();
    }
}
```

```java
/* Example: getEmployees returns an empty list */
List<Employee> employees = getEmployees();
for (Employee e : employees) {
    totalPay += e.getPay();
}

public List<Employee> getEmployees() {
    if (< there are no employees>) {
        return Collections.emptyList();
    }
}
```

When returning an empty list, we can eliminate `null` check, minimize the chance of `NullPointerExceptions` and code looks cleaner.

## Don't Pass Null

We should avoid passing `null` in our code whenever possible.

```java
public class MetricsCalculator {
    public double xProjection(Point p1, Point p2) {
        return (p2.x - p1.x) * 1.5;
    }
}

/* What happens when someone pass null as an argument? */
calculator.xProjection(null, new Point(12, 13));
```

`NullPointerException`. How can we fix it?

```java
/* Example: Create a new exception type and throw it? */
public class MetricsCalculator {
    public double xProjection(Point p1, Point p2) {
        if (p1 == null || p2 == null) {
            throw InvalidArgumentException(*Invalid argument for MetricsCalculator.xProjection*);
        }
        return (p2.x - p1.x) * 1.5;
    }
}
```

When we throw an exception, we have to define a handler of `InvalidArgumentException`.
What should handler do? Is there any good course of action?

```java
/* Example: Use assertions */
public class MetricsCalculator {
    public double xProjection(Point p1, Point p2) {
        assert p1 != null : *p1 should not be null*;
        assert p2 != null : *p2 should not be null*;
        return (p2.x - p1.x) * 1.5;
    }
}
```

Either we use exception or assertions when someone pass `null` we'll still have a runtime error.

The rational approach is to forbid passing `null` by default.

## Conclusion

Clean code is readable, but it must also be robust. These are not conflicting goals. 
We can write robust clean code if we see error handling as a separate concern, 
something that is viewable independently of our main logic. 
To the degree that we are able to do that, we can reason about it independently, 
and we can make great strides in the maintainability of our code.
