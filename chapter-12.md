# Chapter 12 - Emergence

Many of us feel that Kent Beck’s four rules of *Simple Design* are of significant help in creating well-designed software.

According to Kent, a design is “simple” if it follows these rules:
- Runs all the tests
- Contains no duplication
- Expresses the intent of the programmer
- Minimizes the number of classes and methods

The rules are given in order of importance.

### Simple Design Rule 1: Runs All the Tests

First and foremost, a design must produce a system that acts as intended.

A system that is comprehensively tested and passes all of its tests all of the time is a testable system.

Fortunately, making our systems testable pushes us toward a design where our classes are small and single purpose. It’s just easier to test classes that conform to the SRP.

Tight coupling makes it difficult to write tests.

Remarkably, following a simple and obvious rule that says we need to have tests and run them continuously impacts our system’s adherence to the primary OO goals of low coupling and high cohesion.

### Simple Design Rules 2–4: Refactoring

Once we have tests, we are empowered to keep our code and classes clean. We do this by incrementally refactoring the code. For each few lines of code we add, we pause and reflect on the new design. Did we just degrade it? If so, we clean it up and run our tests to demonstrate that we haven’t broken anything. **The fact that we have these tests eliminates the fear that cleaning up the code will break it!**

During this refactoring step, we can apply anything from the entire body of knowledge about good software design. We can increase cohesion, decrease coupling, separate concerns, modularize system concerns, shrink our functions and classes, choose better names, and so on. This is also where we apply the final three rules of simple design: **Eliminate duplication, ensure expressiveness, and minimize the number of classes and methods.**

### No Duplication

Duplication is the primary enemy of a well-designed system. It represents additional work, additional risk, and additional unnecessary complexity. Duplication manifests itself in many forms.

For example, we might have two methods in a collection class:

```java
int size() {}
boolean isEmpty() {}
```

We could have separate implementations for each method. The *isEmpty* method could track a boolean, while *size* could track a counter. Or, we can eliminate this duplication by tying *isEmpty* to the definition of *size*:

```java
boolean isEmpty() {
    return 0 == size();
}
```

Creating a clean system requires the will to eliminate duplication, even in just a few lines of code. For example, consider the following code:

```java
public void scaleToOneDimension(
    float desiredDimension, float imageDimension) {
        if (Math.abs(desiredDimension - imageDimension) < errorThreshold) 
            return;
    float scalingFactor = desiredDimension / imageDimension; 
    scalingFactor = (float)(Math.floor(scalingFactor * 100) * 0.01f);
    
    RenderedOp newImage = ImageUtilities.getScaledImage( 
            image, scalingFactor, scalingFactor);
    image.dispose(); 
    System.gc(); 
    image = newImage;
}
public synchronized void rotate(int degrees) {
    RenderedOp newImage = ImageUtilities.getRotatedImage( 
            image, degrees);
    image.dispose(); 
    System.gc(); 
    image = newImage;
}
```

To keep this system clean, we should eliminate the small amount of duplication between the *scaleToOneDimension* and *rotate* methods:

```java
public void scaleToOneDimension(
    float desiredDimension, float imageDimension) {
        if (Math.abs(desiredDimension - imageDimension) < errorThreshold) 
            return;
    float scalingFactor = desiredDimension / imageDimension; 
    scalingFactor = (float)(Math.floor(scalingFactor * 100) * 0.01f);
    
    replaceImage(ImageUtilities.getScaledImage( 
            image, scalingFactor, scalingFactor));
}

public synchronized void rotate(int degrees) {
    replaceImage(ImageUtilities.getRotatedImage(image, degrees));
}

private void replaceImage(RenderedOp newImage) { 
    image.dispose();
    System.gc();
    image = newImage;
}
```

This “reuse in the small” can cause system complexity to shrink dramatically. Understanding how to achieve reuse in the small is essential to achieving reuse in the large.

The *TEMPLATE METHOD* pattern is a common technique for removing higher-level duplication. For example:

```java
public class VacationPolicy {
    public void accrueUSDivisionVacation() {
        // code to calculate vacation based on hours worked to date 
        // ...
        // code to ensure vacation meets US minimums
        // ...
        // code to apply vaction to payroll record
        // ... 
    }
    public void accrueEUDivisionVacation() {
        // code to calculate vacation based on hours worked to date 
        // ...
        // code to ensure vacation meets EU minimums
        // ...
        // code to apply vaction to payroll record
        // ...
    } 
}
```

The code across *accrueUSDivisionVacation* and *accrueEuropeanDivisionVacation* is largely the same, with the exception of calculating legal minimums. That bit of the algorithm changes based on the employee type.

We can eliminate the obvious duplication by applying the *TEMPLATE METHOD* pattern.

```java
abstract public class VacationPolicy { 
    public void accrueVacation() {
        calculateBaseVacationHours();
        alterForLegalMinimums();
        applyToPayroll();
    }
    
    private void calculateBaseVacationHours() { /* ... */ }; 
    abstract protected void alterForLegalMinimums(); 
    private void applyToPayroll() { /* ... */ };
}

public class USVacationPolicy extends VacationPolicy { 
    @Override protected void alterForLegalMinimums() {
        // US specific logic 
    }
}

public class EUVacationPolicy extends VacationPolicy { 
    @Override protected void alterForLegalMinimums() {
        // EU specific logic 
    }
}
```

The subclasses fill in the “hole” in the *accrueVacation* algorithm, supplying the only bits of
information that are not duplicated.

### Expressive

Code should clearly express the intent of its author. The clearer the author can make the code, the less time others will have to spend understanding it. This will reduce defects and shrink the cost of maintenance.

- You can express yourself by choosing good names.
- You can also express yourself by keeping your functions and classes small.
- You can also express yourself by using standard nomenclature.
- Well-written unit tests are also expressive.
- But the most important way to be expressive is to ***try***.

### Minimal Classes and Methods

Even concepts as fundamental as elimination of duplication, code expressiveness, and the SRP can be taken too far. In an effort to make our classes and methods small, we might create too many tiny classes and methods. So this rule suggests that we also keep our function and class counts low.

High class and method counts are sometimes the result of pointless dogmatism.

Our goal is to keep our overall system small while we are also keeping our functions and classes small. Remember, however, that this rule is the lowest priority of the four rules of Simple Design.

### Conclusion

Is there a set of simple practices that can replace experience? Clearly not. On the other hand, the practices described in this chapter and in this book are a crystallized form of the many decades of experience enjoyed by the authors.
